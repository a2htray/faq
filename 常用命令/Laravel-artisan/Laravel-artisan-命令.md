# Laraval artisan 命令

### 创建 Controller

```bash
# 使用引号，将会创建Wechat文件夹
php artisan make:controller "Wechat\AuthController"
```

### 创建 Migration

```bash
php artisan make:migration create_recommand_stocks
```

### Seeder

```bash
# 创建
php artisan make:seeder UsersTableSeeder

# 执行
php artisan db:seed # 默认--class=DatabaseSeeder
php artisan db:seed --class=UsersTableSeeder # 执行UsersTableSeeder
```
