---
title: 后台权限控制的解决方案--RBAC
date: 2017-09-29 10:28:37
categories: 编程
tags: 
  - laravel
---

## 简介
针对后台不同用户，需要控制用户对模块的操作权限，现采用基于用户角色(RBAC,即Role-Based Access Control)的权限管理方案。  

项目使用 [Zizaco/entrust](https://github.com/Zizaco/entrust/) 扩展包。使用说明参照： https://github.com/Zizaco/entrust/

## 概念
- 角色： 根据用户的不同，设置不同的角色。例如：产品管理员（可以配置/拥有产品相关的权限）
- 权限： 最底层的原子操作权限。例如：产品编辑的权限、产品添加的权限...

<!-- more -->

## 结构
### 数据字典：

**permissions:**

|字段名|类型|说明|
|:---|:---|:---|
|name|string|权限码:格式 `xxxController@action`,系统自动生成|
|parent_id|int|父级权限id|
|display_name|string|权限名称|
|description|string|权限描述|

**roles:**

|字段名|类型|说明|
|:---|:---|:---|
|name|string|角色码|
|display_name|string|角色名称|
|description|string|角色描述|

**permission_role:**

|字段名|类型|说明|
|:---|:---|:---|
|role_id|int| |
|permission_id|int|&nbsp; |

**role_user:**

|字段名|类型|说明|
|:---|:---|:---|
|role_id|int| |
|user_id|int|&nbsp; |

## 后端
### 配置
1. 权限设置  
权限以目录树的形式展示，直观显示权限的层级关系
2. 角色的权限分配  
按权限层级关系的分配，在权限数量比较多的情况下更加方便
3. 基于权限展示不同的菜单  

### 实现
#### 初始化数据填充
> 1. 每次部署执行： `php artisan db:seeder --class=RbacSeeder`
> 2. 权限数据由系统`database/seeds/RbacSeeder.php`文件自动填充，只需在后台修改显示名称即可 
> 3. 权限码（permission 的 name 字段）不允许修改，根据 seeder 文件自动生成

`database/seeds/RbacSeeder.php`文件内容如下：
```php
public function run()
{
    $routes = Route::getRoutes()->getRoutes();
    foreach ($routes as $route) {
        $tmpAction = $route->action['controller']; // 获取系统所有controller
        $str       = "App\\Http\\Controllers\\Admin\\";// 过滤后台需要控制的模块
        if (strpos($tmpAction, $str) === false || strpos($tmpAction, 'create') || strpos($tmpAction, 'edit')) {
            // create/store只保留store, edit/update只保留update
            continue;
        }
        $tmpAction       = str_replace($str, '', $tmpAction);
        $parent          = substr($tmpAction, 0, strpos($tmpAction, '@'));// 初始化层级关系
        $data[$parent][] = [
            'name'         => $tmpAction,
            'display_name' => $this->getNameByAction($tmpAction)
        ];
    }
    foreach ($data as $parent => $item) {
        $tmp  = [
            'name'         => $parent,
            'display_name' => $parent
        ];
        $perm = Permission::where('name', $parent)->first();
        if (!$perm) {
            $perm = Permission::create($tmp);
        }
        foreach ($item as $val) {
            $val['parent_id'] = $perm->id;
            if (!Permission::where('name', $val['name'])->first()) {
                Permission::create($val);
            }
        }
    }
    $this->init();
}

private function getNameByAction($action)
{
    $action = substr($action, strpos($action, '@') + 1);
    switch ($action) {
        case 'index':
            $name = '列表';
            break;
        case 'update':
            $name = '更新';
            break;
        case 'store':
            $name = '添加';
            break;
        case 'show':
            $name = '详情';
            break;
        case 'destroy':
            $name = '删除';
            break;
        default:
            $name = $action;
            break;
    }

    return $name;
}

/**
 * 初始化超级管理员(id为1)权限
 */
private function init()
{
    $role  = [
        'name'         => 'super_admin',
        'display_name' => '超级管理员',
        'description'  => '拥有系统所有权限，并且可以分配其他管理员权限'
    ];
    $role  = Role::create($role);
    $perms = Permission::all();
    $role->attachPermissions($perms);
    User::find(1)->attachRole($role);
}
```
permission最终在数据库的格式如下：(display_name需要到后台页面修改)

|ID|name|parent_id|display_name|description|
|---|---|---|---|---|
|612|ServiceController|0|ServiceController||
|613|ServiceController@index | 612 | 列表||
|614|ServiceController@store | 612 | 添加||
|615|ServiceController@show	| 612 | 详情||
|616|ServiceController@update | 612 | 更新||
|617|ServiceController@destroy | 612 | 删除|&nbsp;|

#### 权限控制
##### 路由中间件
> 思路：`xxxController@action`作为`permission`的*name*字段，然后根据`request`的*actionName*作判断

`app/Http/Middleware/Rbac.php`：

```php
public function handle($request, Closure $next)
{
    $actionName = $request->route()->getActionName();
    $action     = str_replace("App\\Http\\Controllers\\Admin\\", '', $actionName);
    $action     = str_replace("@create", '@store', $action);
    $action     = str_replace("@edit", '@update', $action);
    $user       = $request->user();
    if ($user) {
        // 如果用户已登录
        if (!$user->can($action)) {
            $data = $request->expectsJson() ? [] : view('errors.403');

            return ErrorResponse::create($data, 403)->setErrorCode(ErrorCode::ERR_NOT_ALLOWED);
        }
    }

    return $next($request);
}
```

##### 在菜单中控制
> 对应菜单的`path`上添加对应`xxxController@action`的值进行遍历判断

## 前段
### Permission
```vuejs
<template>
    <div class="types">
        <div class="row">
            <div class="col-md-3">
                <h5>所有类型</h5>
                <br>
                <el-tree
                        :data="data"
                        :props="defaultProps"
                        node-key="id"
                        accordion
                        :default-expanded-keys="[0]"
                        @current-change="show"
                        :expand-on-click-node="true">
                </el-tree>
            </div>
            <div class="col-md-4">
                <h5>添加&更新</h5>
                <br>
                <el-form ref="form" :model="form" :rules="rules" label-width="80px">
                    <el-form-item label="权限标识" prop="name">
                        <el-input :disabled="true" v-model="form.name"></el-input>
                    </el-form-item>
                    <el-form-item label="权限名称" prop="display_name">
                        <el-input v-model="form.display_name"></el-input>
                    </el-form-item>
                    <el-form-item label="上级权限" prop="parent_id">
                        <el-cascader
                                :options="options"
                                change-on-select
                                v-model="selectedOptions"
                                @change="change"
                        ></el-cascader>
                    </el-form-item>
                    <el-form-item label="描述">
                        <el-input type="text" v-model="form.description"></el-input>
                    </el-form-item>
                    <el-form-item>
                        <!--<el-button v-show="form.id == null" type="primary" @click="add">添加</el-button>-->
                        <el-button type="info" @click="update">更新</el-button>
                        <!--<el-button @click="clear">重置表单</el-button>-->
                        <!--权限的添加，是根据seeder文件自动生成，这里只需要修改名称即可-->
                        <!--<el-button v-show="form.id != null" type="warning" @click="remove">删除</el-button>-->
                    </el-form-item>
                </el-form>
            </div>
        </div>
    </div>
</template>
<script>
  import ElementUI from 'element-ui'
  import deepcopy from 'deepcopy'
  import 'element-ui/lib/theme-default/index.css'

  Vue.use(ElementUI)

  export default {
    data () {
      return {
        data: [],
        options: [],
        defaultProps: {
          children: 'children',
          label: 'label'
        },
        selectedOptions: [0],
        form: {
          id: null,
          display_name: '',
          parent_id: [],
          description: ''
        },
        rules: {
          name: [{ required: true, message: '请输入权限标识', trigger: 'blur' }],
          display_name: [{ required: true, message: '请输入权限名称', trigger: 'blur' }],
          parent_id: [{ required: true, message: '请选择上级权限', trigger: 'blur', type: 'array' }]
        }
      }
    },
    created () {
      this.fetchData()
    },
    methods: {
      fetchData () {
        this.$http.get('permission/data').then((ret) => {
          this.data = ret.data.data
          let tmp   = deepcopy(this.data) // Js的数组拷贝真是麻烦，引入第三方库
          // 只保留两个层级
          this.options = tmp.map(function(val) {
            if (val.children != undefined) {
              val.children.map(function(v) {
                delete v.children
                return v
              })
            }
            return val
          })
          this.clear()
        })
      },
      change (val) {
        this.form.parent_id = val
      },
      update () {
        this.$refs['form'].validate((valid) => {
          if (!valid) {
            this.$notify({
              title: '错误',
              message: '信息填写不完整',
              type: 'error'
            })
          } else {
            this.$http.patch('permission/' + this.form.id, {
              name: this.form.name,
              display_name: this.form.display_name,
              parent_id: this.form.parent_id.pop(),
              description: this.form.desc
            }).then(ret => {
              if (ret.data.code == 0) {
                this.$notify({
                  title: '成功',
                  message: '更新成功',
                  type: 'success'
                })
                this.fetchData()
              } else {
                this.$notify({
                  title: '失败',
                  message: '更新失败',
                  type: 'error'
                })
              }
            })
          }
        })
      },
      add () {
        this.$refs['form'].validate((valid) => {
          if (!valid) {
            this.$notify({
              title: '错误',
              message: '信息填写不完整',
              type: 'error'
            })
          } else {
            this.$http.post('permission', {
              name: this.form.name,
              display_name: this.form.display_name,
              parent_id: this.form.parent_id.pop(),
              description: this.form.desc
            }).then(ret => {
              this.$notify({
                title: '成功',
                message: '添加成功',
                type: 'success'
              })
              this.fetchData()
            })
          }
        })
      },
      show (val) {
        this.form.id         = null
        this.selectedOptions = [0]
        this.$http.get('permission/' + val.id).then(ret => {
          let item               = ret.data.data
          this.form.id           = item.id
          this.form.name         = item.name
          this.form.display_name = item.display_name
          this.form.description  = item.description
          this.form.parent_id.push(item.parent_id)
          this.selectedOptions.push(item.parent_id)// 这里只支持2级层级，多级层级的话，需要返回父id的父id..
        })
      },
      clear () {
        this.form.id           = null
        this.form.name         = ''
        this.form.display_name = ''
        this.form.parent_id    = []
        this.form.description  = ''
        this.selectedOptions   = [0]
      },
      remove () {
        if (this.form.id == null) {
          return false
        }
        this.$http.delete('permission/' + this.form.id).then(ret => {
          if (ret.data.code == 0) {
            this.$notify({
              title: '成功',
              message: '删除成功',
              type: 'success'
            })
          } else {
            this.$notify({
              title: '错误',
              message: '删除失败',
              type: 'error'
            })
          }
          this.fetchData()
        })
      }
    }
  }
</script>

<style scoped>
    .el-cascader {
        width: 100%
    }
</style>
```

```php
<?php
namespace App\Http\Controllers\Admin;

use App\Models\Permission;

class PermissionController extends Controller
{
    public function index()
    {
    // 渲染blade页面
    }

    public function getData()
    {
        $types       = Permission::select(\DB::raw('id,name,id as value, parent_id, display_name as label'))->get()->toArray();
        $data        = getTree(0, $types);
        $ret['data'] = [
            [
                'value'    => 0,
                'id'       => 0,
                'label'    => '根目录',
                'children' => $data,
            ],
        ];

        return $this->success($ret);
    }

    public function show($id)
    {
        $data['data'] = Permission::find($id)->toArray();

        return $this->success($data);
    }

    public function store()
    {
        try {
            $return['data'] = Permission::create(request()->all());

            return $this->success($return);
        } catch (\Exception $e) {
            return $this->error(400, '权限已存在');
        }
    }

    public function update($id)
    {
        try {
            $data = request()->all();
            Permission::where('id', $id)->update($data);

            return $this->success(['data' => 'null']);
        } catch (\Exception $e) {
            return $this->error(400, '更新失败！');
        }
    }
}
````

### Role
```vue
<template>
    <div>
        <el-button type="primary" icon="plus" @click="needAction = true">添加角色</el-button>
        <div style="margin-bottom:15px"></div>
        <el-table :data="roles" border style="width: 80%">
            <el-table-column
                    prop="name"
                    label="权限标识">
            </el-table-column>
            <el-table-column
                    prop="display_name"
                    label="角色名称">
            </el-table-column>
            <el-table-column label="操作">
                <template scope="scope">
                    <el-button size="small" icon="edit" @click="handleEdit(scope.row)">编辑</el-button>
                    <el-button size="small" icon="share" type="info" @click="assign(scope.row.id, scope.row.display_name)">分配权限</el-button>
                </template>
            </el-table-column>
        </el-table>

        <el-dialog :title="title" :visible.sync="needAction" size="tiny">
            <el-form :model=" form" ref="form" :rules="rules">
                <el-form-item label="角色代码" label-width="80px" prop="name">
                    <el-input v-model="form.name" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="角色名称" label-width="80px" prop="display_name">
                    <el-input v-model="form.display_name" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="角色描述" label-width="80px">
                    <el-input type="textarea" v-model="form.description"></el-input>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button icon="circle-cross" @click="cancelAddOrUpdate">取 消</el-button>
                <el-button icon="circle-check" type="primary" @click="addOrUpdate">确 定</el-button>
            </div>
        </el-dialog>

        <el-dialog :title="assignTitle" :visible.sync="needAssign" @close="cancelAssign">
            <el-row v-for="(perm, index) in assignForm.perms" :key="perm.id">
                <el-checkbox-group v-model="assignForm.checkedPerms">
                    <el-checkbox style="font-weight: bold" :label="perm.id" @change="handleCheckAllChange(index,perm.id)">{{ perm.label }}：</el-checkbox>
                    <div style="margin: 5px 0;"></div>
                    <el-checkbox :class="{lf: i == 0}" v-for="(item,i) in perm.children" :label="item.id" :key="item.id" @change="handleCheckedChange(index)">{{item.label}}</el-checkbox>
                </el-checkbox-group>
                <hr>
            </el-row>
            <el-row>
                <el-button icon="circle-cross" @click="cancelAssign">取 消</el-button>
                <el-button icon="circle-check" type="primary" @click="doAssign">确 定</el-button>
            </el-row>
        </el-dialog>
    </div>
</template>
<script>
  export default {
    data () {
      return {
        needAction: false,
        needAssign: false,
        id: '',
        action: 'add',
        form: {
          name: '',
          display_name: '',
          description: ''
        },
        rules: {
          name: [
            { required: true, message: '请输入角色代码', trigger: 'blur' }
          ],
          display_name: [
            { required: true, message: '请输入角色名称', trigger: 'blur' }
          ]
        },
        assignTitle: '',
        assignForm: {
          roleId: '',
          checkedPerms: [],
          perms: []
        }
      }
    },
    props: ['roles'],
    computed: {
      title () {
        return this.action == 'add' ? '添加' : '编辑'
      }
    },
    created () {
      this.getPerms()
    },
    methods: {
      cancelAddOrUpdate () {
        this.$refs['form'].resetFields()
        this.form.description = '' // 非必填，reset不了
        this.needAction       = false
        this.action           = 'add'
      },
      addOrUpdate () {
        this.$refs['form'].validate((valid) => {
          if (valid) {
            if (this.action == 'add') {
              this.$http.post('role', this.form).then(ret => {
                if (ret.data.code == 400) {
                  this.$message.error('添加失败！' + ret.data.message)
                } else {
                  this.$message.success('添加成功！')
                  window.location.reload()
                }
              })
            } else {
              this.$http.patch('role/' + this.id, this.form).then(ret => {
                if (ret.data.code != 0) {
                  return this.$message.error('更新失败！')
                }
                this.$message.success('更新成功！')
                this.needAction = false
                this.action     = 'add'
                this.roles.map(val => {
                  if (val.id == this.id) {
                    val.name         = this.form.name
                    val.display_name = this.form.display_name
                    val.description  = this.form.description
                  }
                })
              })
            }
          }
        })
      },
      handleEdit (row) {
        this.form.name         = row.name
        this.form.display_name = row.display_name
        this.form.description  = row.description
        this.id                = row.id
        this.needAction        = true
        this.action            = 'edit'
      },
      getPerms () {
        this.$http.get('permission/data').then((ret) => {
          this.assignForm.perms = ret.data.data[0].children
        })
      },
      assign (roleId, name) {
        this.$http.get('role/' + roleId).then(ret => {
          this.assignForm.checkedPerms = ret.data.data
          this.assignForm.roleId       = roleId
          this.assignTitle             = '当前分配角色：' + name
          this.needAssign              = true
          this.assignForm.perms.map(val => {
            if (this.assignForm.checkedPerms.indexOf(val.id) >= 0) {
              val.checkAll = true
            }
            return val
          })
        })
      },
      cancelAssign () {
        this.needAssign              = false
        this.assignForm.roleId       = ''
        this.assignForm.checkedPerms = []
        this.getPerms()
      },
      doAssign () {
        this.$http.post('role/' + this.assignForm.roleId + '/permission', {
          perms: this.assignForm.checkedPerms
        }).then(ret => {
          if (ret.data.code == 0) {
            this.$message.success('分配成功！')
            window.location.reload()
          } else {
            this.$message.error('分配失败！')
          }
        })
      },
      handleCheckAllChange (index, id) {
        let event    = window.event
        let subPerms = this.assignForm.perms[index].children
        if (event.target.checked) {
          // 全选
          for (let i = 0; i < subPerms.length; i++) {
            // 添加所有子节点
            this.assignForm.checkedPerms.push(subPerms[i].id)
          }
        } else {
          // 全不选
          let tmp = []
          for (let i = 0; i < subPerms.length; i++) {
            tmp.push(subPerms[i].id)
          }
          this.assignForm.checkedPerms = this.assignForm.checkedPerms.filter(val => {
            return tmp.indexOf(val) < 0
          })
        }
      },
      handleCheckedChange (index) {
        let children  = this.assignForm.perms[index].children
        let parent_id = this.assignForm.perms[index].id
        for (let i = 0; i < children.length; i++) {
          let key = this.assignForm.checkedPerms.indexOf(children[i].id)
          let tmp = this.assignForm.checkedPerms.indexOf(parent_id)
          if (key < 0 && tmp > -1) {
            // 不是全选
            this.assignForm.checkedPerms.splice(tmp, 1)
            return
          } else if (tmp < 0) {
            // 全选
            this.assignForm.checkedPerms.push(parent_id)
          }
        }
      }
    }
  }
</script>
<style>
    .el-transfer-panel {
        height: 350px;
    }

    .el-tag {
        margin: 0 2px;
    }

    .lf {
        margin-left: 50px
    }
</style>
```
```php
<?php
namespace App\Http\Controllers\Admin;

use App\Models\Permission;
use App\Models\PermissionRole;
use App\Models\Role;

class RoleController extends Controller
{
    public function index()
    {
        $roles = Role::get();

        return [
            'template' => 'admin.system.role',
            'roles'    => $roles,
        ];
    }

    public function show($id)
    {
        $return['data'] = PermissionRole::where('role_id', $id)->pluck('permission_id')->toArray();

        return $this->success($return);
    }

    public function update($id)
    {
        try {
            Role::where('id', $id)->update(request()->all());

            return $this->success(['data' => 'null']);
        } catch (\Exception $e) {
            return $this->error(400, '更新失败！');
        }
    }

    public function store()
    {
        try {
            $return['data'] = Role::create(request()->all());

            return $this->success($return);
        } catch (\Exception $e) {
            return $this->error(400, '角色已存在');
        }
    }

    public function permission($id)
    {
        try {
            PermissionRole::where('role_id', $id)->delete();// 先清空权限
            $perms = Permission::whereIn('id', request()->get('perms'))->get();
            $role  = Role::find($id);
            $role->attachPermissions($perms);
            \Cache::tags(config('entrust.permission_role_table'))->flush();// 手动清空缓存

            return $this->success(['data' => null]);
        } catch (\Exception $e) {
            return $this->error(400, '分配失败！');
        }
    }

    public function data()
    {
        $return['data'] = Role::get()->toArray();

        return $this->success($return);
    }
}
```

### User
```javascript
<template>
    <div>
        <el-button type="primary" icon="plus" @click="needAction = true">添加用户</el-button>
        <div style="margin-bottom:15px"></div>
        <el-table :data="users" border>
            <el-table-column
                    prop="name"
                    label="用户名">
            </el-table-column>
            <el-table-column
                    prop="email"
                    label="邮箱">
            </el-table-column>
            <el-table-column
                    prop="roles"
                    label="拥有角色">
                <template scope="scope">
                    <el-tag v-for="(role,index) in scope.row.roles" :key="index" type="info" close-transition>{{role.display_name}}</el-tag>
                </template>
            </el-table-column>
            <el-table-column label="操作">
                <template scope="scope">
                    <el-button size="small" icon="edit" @click="handleEdit(scope.row)">编辑</el-button>
                    <el-button size="small" icon="share" type="info" @click="assign(scope.row.id, scope.row.name)">分配角色</el-button>
                </template>
            </el-table-column>
        </el-table>

        <el-dialog :title="title" :visible.sync="needAction" size="tiny" @close="cancelAddOrUpdate">
            <el-form :model=" form" ref="form" :rules="rules">
                <el-form-item label="用户名" label-width="80px" prop="name">
                    <el-input v-model="form.name" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="邮箱" label-width="80px" prop="email">
                    <el-input v-model="form.email" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="密码" label-width="80px" prop="password">
                    <el-input type="password" v-model="form.password" auto-complete="off"></el-input>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button icon="circle-cross" @click="cancelAddOrUpdate">取 消</el-button>
                <el-button icon="circle-check" type="primary" @click="addOrUpdate">确 定</el-button>
            </div>
        </el-dialog>

        <el-dialog :title="assignTitle" :visible.sync="needAssign" size="tiny">
            <el-checkbox-group v-model="assignForm.checkedRoles">
                <ul>
                    <li v-for="role in assignForm.roles" :key="role.id">
                        <el-checkbox :label="role.id">{{ role.display_name }}</el-checkbox>
                    </li>
                </ul>
            </el-checkbox-group>
            <div slot="footer" class="dialog-footer">
                <el-button icon="circle-cross" @click="cancelAssign">取 消</el-button>
                <el-button icon="circle-check" type="primary" @click="doAssign">确 定</el-button>
            </div>
        </el-dialog>
    </div>
</template>
<script>
  export default {
    data () {
      return {
        needAction: false,
        needAssign: false,
        id: '',
        action: 'add',
        form: {
          name: '',
          email: '',
          password: ''
        },
        rules: {
          name: [
            { required: true, message: '请输入用户名', trigger: 'blur' }
          ],
          email: [
            { required: true, message: '请输入邮箱', trigger: 'blur' }
          ],
          password: [
            { required: true, message: '请输入密码', trigger: 'blur' },
            { min: 3, message: '最小3个字符', trigger: 'blur' }
          ]
        },
        assignTitle: '',
        assignForm: {
          userId: '',
          checkedRoles: [],
          roles: []
        }
      }
    },
    props: ['users'],
    computed: {
      title () {
        return this.action == 'add' ? '添加' : '编辑'
      }
    },
    created () {
      this.getRoles()
    },
    methods: {
      cancelAddOrUpdate () {
        this.form.name     = ''
        this.form.email    = ''
        this.form.password = ''
        this.needAction    = false
        this.action        = 'add'
      },
      addOrUpdate () {
        this.$refs['form'].validate((valid) => {
          if (valid) {
            if (this.action == 'add') {
              this.$http.post('user', this.form).then(ret => {
                if (ret.data.code == 400) {
                  this.$message.error('添加失败！' + ret.data.message)
                } else {
                  this.$message.success('添加成功！')
                  window.location.reload()
                }
              })
            } else {
              this.$http.patch('user/' + this.id, this.form).then(ret => {
                if (ret.data.code != 0) {
                  return this.$message.error('更新失败！')
                }
                this.$message.success('更新成功！')
                this.needAction = false
                this.action     = 'add'
                this.users.map(val => {
                  if (val.id == this.id) {
                    val.name  = this.form.name
                    val.email = this.form.email
                  }
                })
              })
            }
          }
        })
      },
      handleEdit (row) {
        this.form.name  = row.name
        this.form.email = row.email
        this.id         = row.id
        this.needAction = true
        this.action     = 'edit'
      },
      getRoles () {
        this.$http.get('role/data').then((ret) => {
          this.assignForm.roles = ret.data.data
        })
      },
      assign (userId, name) {
        this.$http.get('user/' + userId).then(ret => {
          console.log(ret)
          this.assignForm.checkedRoles = ret.data.data
          this.assignForm.userId       = userId
          this.assignTitle             = '当前分配用户：' + name
          this.needAssign              = true
        })
      },
      cancelAssign () {
        this.needAssign              = false
        this.assignForm.userId       = ''
        this.assignForm.checkedRoles = []
      },
      doAssign () {
        this.$http.post('user/' + this.assignForm.userId + '/role', {
          roles: this.assignForm.checkedRoles
        }).then(ret => {
          if (ret.data.code == 0) {
            this.$message.success('分配成功！')
            window.location.reload()
          } else {
            this.$message.error('分配失败！')
          }
        })
      }
    }
  }
</script>
<style>
    .el-transfer-panel {
        height: 350px;
    }

    .el-tag {
        margin: 0 2px;
    }
</style>
```
UserController.php
```php
<?php

namespace App\Http\Controllers\Admin;

use App\Models\Role;
use App\Models\RoleUser;
use App\Models\User;

class UserController extends Controller
{

    public function index()
    {
        $users = User::with('roles')->get();

        return [
            'template' => 'admin.system.user',
            'users'    => $users,
        ];
    }

    public function store()
    {
        $data             = request()->all();
        $data['password'] = bcrypt($data['password']);
        try {
            $return['data'] = User::create($data);

            return $this->success($return);
        } catch (\Exception $e) {
            return $this->error(400, '用户已存在');
        }
    }

    public function update($id)
    {

        try {
            $data             = request()->all();
            $data['password'] = bcrypt($data['password']);

            User::where('id', $id)->update($data);

            return $this->success(['data' => 'null']);
        } catch (\Exception $e) {
            return $this->error(400, '更新失败！');
        }
    }

    public function show($id)
    {
        $user           = User::find($id);
        $return['data'] = $user->roles->pluck('id');

        return $this->success($return);
    }

    public function role($id)
    {
        try {
            RoleUser::where('user_id', $id)->delete();// 先清空角色
            $roles = Role::whereIn('id', request()->get('roles'))->get();
            $user  = User::find($id);

            $user->attachRoles($roles);
            \Cache::tags(config('entrust.role_user_table'))->flush();

            return $this->success(['data' => null]);
        } catch (\Exception $e) {
            return $this->error(400, '分配失败！');
        }
    }
}
```
