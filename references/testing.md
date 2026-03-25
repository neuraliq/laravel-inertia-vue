# Testing Inertia Pages

## Pest (Server-Side)

```php
// tests/Feature/Users/IndexTest.php
use App\Models\User;

it('renders the users index page', function () {
    $users = User::factory(3)->create();

    $this->actingAs(User::factory()->create())
        ->get('/users')
        ->assertOk()
        ->assertInertia(fn ($page) => $page
            ->component('Users/Index')
            ->has('users.data', 3)
            ->has('users.data.0', fn ($user) => $user
                ->has('id')
                ->has('name')
                ->has('email')
                ->missing('password')
            )
        );
});

it('filters users by search query', function () {
    User::factory()->create(['name' => 'John Doe']);
    User::factory()->create(['name' => 'Jane Smith']);

    $this->actingAs(User::factory()->create())
        ->get('/users?search=John')
        ->assertInertia(fn ($page) => $page
            ->component('Users/Index')
            ->has('users.data', 1)
            ->where('users.data.0.name', 'John Doe')
            ->where('filters.search', 'John')
        );
});

it('creates a user and redirects', function () {
    $this->actingAs(User::factory()->create())
        ->post('/users', [
            'name' => 'New User',
            'email' => 'new@example.com',
            'password' => 'password123',
        ])
        ->assertRedirect('/users')
        ->assertSessionHas('success', 'User created.');

    $this->assertDatabaseHas('users', ['email' => 'new@example.com']);
});

it('validates user creation', function () {
    $this->actingAs(User::factory()->create())
        ->post('/users', [
            'name' => '',
            'email' => 'invalid',
        ])
        ->assertInertia(fn ($page) => $page
            ->has('errors.name')
            ->has('errors.email')
        );
});
```

## Testing Shared Data

```php
it('shares auth user data', function () {
    $user = User::factory()->create();

    $this->actingAs($user)
        ->get('/dashboard')
        ->assertInertia(fn ($page) => $page
            ->where('auth.user.id', $user->id)
            ->where('auth.user.name', $user->name)
        );
});

it('shares flash messages after action', function () {
    $this->actingAs(User::factory()->create())
        ->followingRedirects()
        ->post('/users', User::factory()->raw())
        ->assertInertia(fn ($page) => $page
            ->where('flash.success', 'User created.')
        );
});
```

## Testing Authorization

```php
it('returns 403 for unauthorized users', function () {
    $regularUser = User::factory()->create();

    $this->actingAs($regularUser)
        ->get('/admin/settings')
        ->assertForbidden();
});

it('allows admins to access settings', function () {
    $admin = User::factory()->admin()->create();

    $this->actingAs($admin)
        ->get('/admin/settings')
        ->assertOk()
        ->assertInertia(fn ($page) => $page
            ->component('Admin/Settings')
        );
});
```
