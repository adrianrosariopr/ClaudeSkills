# Workflow: Set Up Team-Based Permissions

<required_reading>
**Read these reference files NOW:**
1. references/multi-tenancy.md
2. references/architecture.md
3. references/caching.md
</required_reading>

<process>
## Step 1: Enable Teams in Config

```php
// config/permission.php
'teams' => true,
```

## Step 2: Add Team Column to Permission Tables

If tables already exist, create migration:

```bash
php artisan make:migration add_team_id_to_permission_tables
```

```php
// database/migrations/xxxx_add_team_id_to_permission_tables.php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
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

    public function down(): void
    {
        Schema::table('model_has_roles', function (Blueprint $table) {
            $table->dropColumn('team_id');
        });

        Schema::table('model_has_permissions', function (Blueprint $table) {
            $table->dropColumn('team_id');
        });
    }
};
```

Run migration:
```bash
php artisan migrate
```

## Step 3: Create Team Model

```bash
php artisan make:model Team -m
```

```php
// app/Models/Team.php
namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsToMany;

class Team extends Model
{
    protected $fillable = ['name', 'slug'];

    public function users(): BelongsToMany
    {
        return $this->belongsToMany(User::class)->withTimestamps();
    }
}
```

```php
// database/migrations/xxxx_create_teams_table.php
public function up(): void
{
    Schema::create('teams', function (Blueprint $table) {
        $table->id();
        $table->string('name');
        $table->string('slug')->unique();
        $table->timestamps();
    });

    Schema::create('team_user', function (Blueprint $table) {
        $table->id();
        $table->foreignId('team_id')->constrained()->cascadeOnDelete();
        $table->foreignId('user_id')->constrained()->cascadeOnDelete();
        $table->timestamps();

        $table->unique(['team_id', 'user_id']);
    });
}
```

## Step 4: Update User Model

```php
// app/Models/User.php
namespace App\Models;

use Illuminate\Database\Eloquent\Relations\BelongsToMany;
use Illuminate\Database\Eloquent\Relations\BelongsTo;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Spatie\Permission\Traits\HasRoles;

class User extends Authenticatable
{
    use HasRoles;

    protected $fillable = [
        'name', 'email', 'password', 'current_team_id',
    ];

    public function teams(): BelongsToMany
    {
        return $this->belongsToMany(Team::class)->withTimestamps();
    }

    public function currentTeam(): BelongsTo
    {
        return $this->belongsTo(Team::class, 'current_team_id');
    }
}
```

Add migration for current_team_id:
```php
Schema::table('users', function (Blueprint $table) {
    $table->foreignId('current_team_id')->nullable()->constrained('teams')->nullOnDelete();
});
```

## Step 5: Create Team Context Middleware

```php
// app/Http/Middleware/SetTeamContext.php
namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Spatie\Permission\PermissionRegistrar;
use Symfony\Component\HttpFoundation\Response;

class SetTeamContext
{
    public function handle(Request $request, Closure $next): Response
    {
        $user = $request->user();

        if ($user && $user->current_team_id) {
            app(PermissionRegistrar::class)->setPermissionsTeamId($user->current_team_id);
        }

        return $next($request);
    }
}
```

## Step 6: Register Middleware with Correct Priority

```php
// bootstrap/app.php
use App\Http\Middleware\SetTeamContext;
use Spatie\Permission\Middleware\PermissionMiddleware;
use Spatie\Permission\Middleware\RoleMiddleware;
use Spatie\Permission\Middleware\RoleOrPermissionMiddleware;

return Application::configure(basePath: dirname(__DIR__))
    ->withMiddleware(function (Middleware $middleware) {
        $middleware->alias([
            'team' => SetTeamContext::class,
            'role' => RoleMiddleware::class,
            'permission' => PermissionMiddleware::class,
            'role_or_permission' => RoleOrPermissionMiddleware::class,
        ]);

        // CRITICAL: Team context must be set before permission checks
        $middleware->priority([
            SetTeamContext::class,
            PermissionMiddleware::class,
            RoleMiddleware::class,
            \Illuminate\Routing\Middleware\SubstituteBindings::class,
        ]);

        // Apply team middleware to web routes
        $middleware->web(append: [
            SetTeamContext::class,
        ]);
    })
    ->create();
```

## Step 7: Create Team Roles Seeder

```php
// database/seeders/TeamRolesSeeder.php
namespace Database\Seeders;

use App\Models\Team;
use Illuminate\Database\Seeder;
use Spatie\Permission\Models\Permission;
use Spatie\Permission\Models\Role;
use Spatie\Permission\PermissionRegistrar;

class TeamRolesSeeder extends Seeder
{
    public function run(): void
    {
        app()[PermissionRegistrar::class]->forgetCachedPermissions();

        // Create global permissions (shared across all teams)
        $permissions = [
            'view projects', 'create projects', 'edit projects', 'delete projects',
            'view members', 'invite members', 'remove members',
            'manage team settings',
        ];

        foreach ($permissions as $name) {
            Permission::firstOrCreate(['name' => $name]);
        }

        // Create roles for each team
        Team::all()->each(function (Team $team) {
            app(PermissionRegistrar::class)->setPermissionsTeamId($team->id);

            // Team owner - all permissions
            $owner = Role::firstOrCreate([
                'name' => 'owner',
                'team_id' => $team->id,
            ]);
            $owner->syncPermissions(Permission::all());

            // Team admin
            $admin = Role::firstOrCreate([
                'name' => 'admin',
                'team_id' => $team->id,
            ]);
            $admin->syncPermissions([
                'view projects', 'create projects', 'edit projects',
                'view members', 'invite members',
            ]);

            // Team member
            $member = Role::firstOrCreate([
                'name' => 'member',
                'team_id' => $team->id,
            ]);
            $member->syncPermissions(['view projects', 'view members']);
        });
    }
}
```

## Step 8: Implement Team Switching

```php
// app/Http/Controllers/TeamController.php
namespace App\Http\Controllers;

use App\Models\Team;
use Illuminate\Http\Request;

class TeamController extends Controller
{
    public function switch(Request $request, Team $team)
    {
        $user = $request->user();

        // Verify user belongs to team
        if (!$user->teams->contains($team)) {
            abort(403, 'You are not a member of this team.');
        }

        // Update current team
        $user->update(['current_team_id' => $team->id]);

        // Clear cached relations (important!)
        $user->unsetRelation('roles');
        $user->unsetRelation('permissions');

        return redirect()->back()->with('success', "Switched to {$team->name}");
    }
}
```

```php
// routes/web.php
Route::middleware('auth')->group(function () {
    Route::post('/teams/{team}/switch', [TeamController::class, 'switch'])
        ->name('teams.switch');
});
```

## Step 9: Invite User to Team

```php
// app/Services/TeamService.php
namespace App\Services;

use App\Models\Team;
use App\Models\User;
use Spatie\Permission\PermissionRegistrar;

class TeamService
{
    public function addMember(Team $team, User $user, string $role = 'member'): void
    {
        // Add to team
        $team->users()->attach($user);

        // Set first team as current if user has none
        if (!$user->current_team_id) {
            $user->update(['current_team_id' => $team->id]);
        }

        // Set team context and assign role
        app(PermissionRegistrar::class)->setPermissionsTeamId($team->id);
        $user->assignRole($role);
    }

    public function removeMember(Team $team, User $user): void
    {
        // Set team context
        app(PermissionRegistrar::class)->setPermissionsTeamId($team->id);

        // Remove all roles in this team
        $user->syncRoles([]);

        // Remove from team
        $team->users()->detach($user);

        // Clear current team if it was this one
        if ($user->current_team_id === $team->id) {
            $newTeam = $user->teams()->first();
            $user->update(['current_team_id' => $newTeam?->id]);
        }
    }
}
```

## Step 10: Team-Aware Views

```blade
{{-- Team switcher --}}
@if(auth()->user()->teams->count() > 1)
    <div class="team-switcher">
        <span>Team: {{ auth()->user()->currentTeam->name }}</span>
        <ul>
            @foreach(auth()->user()->teams as $team)
                @if($team->id !== auth()->user()->current_team_id)
                    <li>
                        <form action="{{ route('teams.switch', $team) }}" method="POST">
                            @csrf
                            <button type="submit">{{ $team->name }}</button>
                        </form>
                    </li>
                @endif
            @endforeach
        </ul>
    </div>
@endif

{{-- Permission checks work per-team --}}
@can('invite members')
    <a href="{{ route('teams.members.invite') }}">Invite Member</a>
@endcan
```

## Step 11: Verify Setup

```bash
php artisan tinker
```

```php
use App\Models\User;
use App\Models\Team;
use Spatie\Permission\PermissionRegistrar;

// Create test team
$team = Team::create(['name' => 'Acme Corp', 'slug' => 'acme']);

// Create user and add to team
$user = User::factory()->create();
$team->users()->attach($user);
$user->update(['current_team_id' => $team->id]);

// Set team context
app(PermissionRegistrar::class)->setPermissionsTeamId($team->id);

// Assign team role
$user->assignRole('admin');

// Verify
$user->getRoleNames();  // ['admin']
$user->can('edit projects');  // true (if admin has this permission)
```
</process>

<anti_patterns>
Avoid:
- Forgetting to set team context before permission operations
- Not unsetting relations when switching teams
- Wrong middleware priority (SubstituteBindings before team context)
- Creating roles without team_id when teams mode is enabled
</anti_patterns>

<success_criteria>
Teams setup complete when:
- [ ] Teams enabled in config
- [ ] Migration adds team_id columns
- [ ] Team model created with user relationship
- [ ] User model has teams and currentTeam relationships
- [ ] SetTeamContext middleware registered with correct priority
- [ ] Team roles seeded for existing teams
- [ ] Team switching works correctly
- [ ] Permission checks respect team context
- [ ] Users can have different roles in different teams
</success_criteria>
