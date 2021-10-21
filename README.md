# Inertia SSR Page Cache

## 1. Setup Official SSR

https://inertiajs.com/server-side-rendering

## 2. Install Laravel Page Cache

```bash
composer require silber/page-cache
```

Register middleware.

Update Nginx config:

```nginx
    set $response_ext '.html';
    set $vary '';
    if ($http_x_inertia = 'true') {
        set $response_ext '.json';
        set $vary 'Accept';
    }

    location = / {
        try_files /page-cache/pc__index__pc$response_ext /index.php?$query_string;
        add_header X-Inertia $http_x_inertia;
        add_header Vary $vary;
    }

    location / {
        try_files $uri $uri/ /page-cache/$uri$response_ext /index.php?$query_string;
        add_header X-Inertia $http_x_inertia;
        add_header Vary $vary;
    }
```

More info see: https://github.com/JosephSilber/page-cache

### 3. Generate Pages

```bash
php artisan make:test CreatePagesCacheTest
```

```php
<?php

namespace Tests\Feature;

use App\Http\Kernel;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Silber\PageCache\Middleware\CacheResponse;
use Tests\TestCase;

class CreatePagesCacheTest extends TestCase
{
    use RefreshDatabase;

    public function test_should_cache_pages()
    {
        $this->app[Kernel::class]->appendMiddlewareToGroup('web', CacheResponse::class);

        $inertiaHeaders = [
            'X-Inertia' => true,
            'X-Inertia-Version' => hash_file('md5', public_path('mix-manifest.json')),
        ];

        $this->get('/')->assertStatus(200);
        $this->get('/', $inertiaHeaders)->assertStatus(200);
        $this->get('/about')->assertStatus(200);
        $this->get('/about', $inertiaHeaders)->assertStatus(200);
    }
}

```
