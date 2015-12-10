# Обнаруженные ошибки по проекту youfhe.ru

## Error

* `/user/user/profile` - нельзя именить данные

## Done

* `/home/admin/web/youfhe.ru/public_html/youfhe/application/config/aliases.php` - не верные данные

```
return[
    '@core' => '/home/admin/web/youfhe.ru/public_html/youfhe/application/modules/core/models/../',
    '@DefaultTheme' => '/home/admin/web/youfhe.ru/public_html/youfhe/application/extensions/DefaultTheme',
    '@shop' => '/home/admin/web/youfhe.ru/public_html/youfhe/application/backgroundtasks/models/../',
    '@category' => '/shop/product/list',
    '@product' => '/shop/product/show',
    '@user' => '/home/admin/web/youfhe.ru/public_html/youfhe/application/modules/user/models/../',
    '@article' => '/page/page/show',
    '@articles' => '/page/page/list',
];
```
____ 
