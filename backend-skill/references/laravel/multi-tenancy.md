<overview>
Implementing team-based (multi-tenant) permissions with Spatie. Each team has its own roles and permissions scope.
</overview>

<enable_teams>
**Step 1: Enable teams in config**

```php
// config/permission.php
'teams' => true,
```

**Step 2: Add team foreign key to tables**

```php
Schema::table('model_has_roles', function (Blueprint $table) {
    $table->unsignedBigInteger('team_id')->nullable()->after('role_id');
    $table->index('team_id');
});

Schema::table('model_has_permissions', function (Blueprint $table) {
    $table->unsignedBigInteger('team_id')->nullable()->after('permission_id');
    $table->index('team_id');
});
```
</enable_teams>

<team_middleware>
**Create middleware to set team context:**

```php
// app/Http/Middleware/SetTeamContext.php
class SetTeamContext
{
    public function handle(Request $request, Closure $next)
    {
        $teamId = session('current_team_id')
            ?? $request->route('team')?->id
            ?? $request->user()?->current_team_id;

        if ($teamId) {
            app(PermissionRegistrar::class)->setPermissionsTeamId($teamId);
        }

        return $next($request);
    }
}
```

**Register middleware (Laravel 12):**

```php
// bootstrap/app.php
->withMiddleware(function (Middleware $middleware) {
    $middleware->alias([
        'team' => \App\Http\Middleware\SetTeamContext::class,
    ]);

    $middleware->priority([
        \App\Http\Middleware\SetTeamContext::class,
        \Spatie\Permission\Middleware\PermissionMiddleware::class,
    ]);
})
```
</team_middleware>

<team_scoped_operations>
**Create team-scoped roles:**

```php
// Set team context
app(PermissionRegistrar::class)->setPermissionsTeamId($team->id);

// Create role for this team
$role = Role::create(['name' => 'editor', 'team_id' => $team->id]);

// Assign role (scoped to current team)
$user->assignRole('editor');
```

**Global roles (team_id = null):**

```php
app(PermissionRegistrar::class)->setPermissionsTeamId(null);
$globalRole = Role::create(['name' => 'super-admin']);
```
</team_scoped_operations>

<switching_teams>
```php
public function switchTeam(Team $team)
{
    if (!$user->teams->contains($team)) {
        abort(403);
    }

    session(['current_team_id' => $team->id]);

    // IMPORTANT: Clear cached roles/permissions
    $user->unsetRelation('roles');
    $user->unsetRelation('permissions');

    return redirect()->back();
}
```
</switching_teams>

<team_model_example>
```php
// app/Models/Team.php
class Team extends Model
{
    public function users(): BelongsToMany
    {
        return $this->belongsToMany(User::class);
    }
}

// app/Models/User.php
class User extends Authenticatable
{
    use HasRoles;

    public function teams(): BelongsToMany
    {
        return $this->belongsToMany(Team::class);
    }

    public function currentTeam()
    {
        return $this->belongsTo(Team::class, 'current_team_id');
    }
}
```
</team_model_example>

<inviting_users>
```php
public function inviteToTeam(User $user, Team $team, string $role): void
{
    $team->users()->attach($user);

    app(PermissionRegistrar::class)->setPermissionsTeamId($team->id);
    $user->assignRole($role);
}

public function removeFromTeam(User $user, Team $team): void
{
    app(PermissionRegistrar::class)->setPermissionsTeamId($team->id);
    $user->syncRoles([]);
    $team->users()->detach($user);
}
```
</inviting_users>

<common_issues>
**Issue: Wrong team context in queue jobs**

Queue jobs don't have session. Pass team_id explicitly:

```php
public function __construct(public int $teamId) {}

public function handle(): void
{
    app(PermissionRegistrar::class)->setPermissionsTeamId($this->teamId);
    // ...
}
```
</common_issues>
