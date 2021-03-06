# django + vue 动态执行计划任务
包含 `用户`、`角色`、`菜单`、`权限` 管理， 使用django-celery-beat实现动态配置任务，

- 后端model参考: [loonflow](https://github.com/blackholll/loonflow), 非常不错的一个项目
- 前端设计参考: [花裤衩 vue-element-admin](https://github.com/PanJiaChen/vue-element-admin), 大神作品没得说
## 开发环境
### 后端
安装依赖
```bash
cd backend
pip install -r requirements.txt
```

初始化系统
- 生成管理员账号 `admin 123456`
```bash
python manage.py migrate
python manage.py init_sys
python manage.py init_celery
```

celery运行
```bash
celery -A celery_tasks.celery worker -B -l info 
```

运行
```bash
python manage.py runserver
```

### 前端
```bash
cd frontend
npm install
npm run dev
```

## 开始使用
使用 `admin` 登录
### 给所有角色分配工作流权限
![role](https://github.com/itimor/one-workflow/raw/master/gifs/role.png)

### 分配菜单 和 数据 权限
![role_edit](https://github.com/itimor/one-workflow/raw/master/gifs/role_edit.png)
