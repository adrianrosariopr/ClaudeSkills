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

If not already migrated, create migration:

```php
// database/migrations/xxxx_add_teams_to_permission_tables.php
public function up(): void
{
    Schema::table('model_has_roles', function (Blueprint $table) {
        $table->unsignedBigInteger('team_id')->nullable()->after('role_id');
        $table->index('team_id');
    });

    Schema::table('model_has_permissions', function (Blueprint $table) {
        $table->unsignedBigInteger('team_id')->nullable()->after('permission_id');
        $table->index('team_id');
    });
}
```
</enable_teams>

<team_middleware>
**Create middleware to set team context:**

```php
// app/Http/Middleware/SetTeamContext.php
namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Spatie\Permission\PermissionRegistrar;

class SetTeamContext
{
    public function handle(Request $request, Closure $next)
    {
        // Get team from session, route, or user
        $teamId = session('current_team_id')
            ?? $request->route('team')?->id
            ?? $request->user()?->current_team_id;

        if ($teamId) {
            app(PermissionRegistrar::class)->setPermissionsTeamId($teamId);
        }

        return $next($response);
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

    // Ensure team middleware runs before permission checks
    $middleware->priority([
        \App\Http\Middleware\SetTeamContext::class,
        \Spatie\Permission\Middleware\PermissionMiddleware::class,
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
    ]);
})
```
</team_middleware>

<team_scoped_operations>
**Create team-scoped roles:**

```php
use Spatie\Permission\PermissionRegistrar;

// Set team context
app(PermissionRegistrar::class)->setPermissionsTeamId($team->id);

// Create role for this team
$role = Role::create(['name' => 'editor']);

// Or create with explicit team_id
$role = Role::create([
    'name' => 'editor',
    'team_id' => $team->id,
]);
```

**Global roles (team_id = null):**

```php
// Without team context set
app(PermissionRegistrar::class)->setPermissionsTeamId(null);

// This creates a global role
$globalRole = Role::create(['name' => 'super-admin']);
```

**Assign team-scoped role:**

```php
// Set team context first
app(PermissionRegistrar::class)->setPermissionsTeamId($team->id);

// Assign role (scoped to current team)
$user->assignRole('editor');

// User now has 'editor' role ONLY for this team
```
</team_scoped_operations>

<switching_teams>
**Switching active team:**

```php
// In controller
public function switchTeam(Team $team)
{
    // Verify user belongs to team
    if (!$user->teams->contains($team)) {
        abort(403);
    }

    // Update session
    session(['current_team_id' => $team->id]);

    // IMPORTANT: Clear cached roles/permissions
    $user->unsetRelation('roles');
    $user->unsetRelation('permissions');

    return redirect()->back();
}
```

**Clear relations when switching:**

```php
// After changing team context
$user->unsetRelation('roles');
$user->unsetRelation('permissions');

// Now queries reflect new team
$user->getRoleNames(); // Roles for new team
```
</switching_teams>

<team_permission_checks>
**Check permission in team context:**

```php
// Set team context
app(PermissionRegistrar::class)->setPermissionsTeamId($team->id);

// Check permission (scoped to team)
$user->hasPermissionTo('edit articles');

// Check role (scoped to team)
$user->hasRole('editor');
```

**Blade with team context:**

Team context should be set in middleware before views render.

```blade
@can('edit articles')
    {{-- User has permission in current team --}}
@endcan
```
</team_permission_checks>

<team_model_example>
```php
// app/Models/Team.php
namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsToMany;

class Team extends Model
{
    protected $fillable = ['name'];

    public function users(): BelongsToMany
    {
        return $this->belongsToMany(User::class);
    }
}
```

```php
// app/Models/User.php
use Spatie\Permission\Traits\HasRoles;

class User extends Authenticatable
{
    use HasRoles;

    protected $fillable = ['name', 'email', 'password', 'current_team_id'];

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

<inviting_users_to_team>
```php
// Invite user to team with role
public function inviteToTeam(User $user, Team $team, string $role): void
{
    // Add user to team
    $team->users()->attach($user);

    // Set team context
    app(PermissionRegistrar::class)->setPermissionsTeamId($team->id);

    // Assign role in team context
    $user->assignRole($role);
}

// Remove user from team
public function removeFromTeam(User $user, Team $team): void
{
    // Set team context
    app(PermissionRegistrar::class)->setPermissionsTeamId($team->id);

    // Remove all roles in this team
    $user->syncRoles([]);

    // Remove from team
    $team->users()->detach($user);
}
```
</inviting_users_to_team>

<seeding_team_permissions>
```php
// database/seeders/TeamPermissionsSeeder.php
class TeamPermissionsSeeder extends Seeder
{
    public function run(): void
    {
        app()[PermissionRegistrar::class]->forgetCachedPermissions();

        // Create global permissions (shared across teams)
        $permissions = ['view articles', 'create articles', 'edit articles', 'delete articles'];
        foreach ($permissions as $perm) {
            Permission::create(['name' => $perm]);
        }

        // For each team, create team-specific roles
        Team::all()->each(function (Team $team) {
            app(PermissionRegistrar::class)->setPermissionsTeamId($team->id);

            $admin = Role::create(['name' => 'admin', 'team_id' => $team->id]);
            $admin->givePermissionTo(Permission::all());

            $member = Role::create(['name' => 'member', 'team_id' => $team->id]);
            $member->givePermissionTo(['view articles', 'create articles']);
        });
    }
}
```
</seeding_team_permissions>

<common_issues>
**Issue: "Model does not have relationship named team"**

Spatie doesn't create the Team model. You must create it and configure relationships.

**Issue: Wrong team context in queue jobs**

Queue jobs don't have session. Pass team_id explicitly:

```php
// In job
public function __construct(public int $teamId) {}

public function handle(): void
{
    app(PermissionRegistrar::class)->setPermissionsTeamId($this->teamId);
    // ...
}
```

**Issue: 404 instead of 403**

Ensure middleware priority puts team context before permission checks and before SubstituteBindings.
</common_issues>
