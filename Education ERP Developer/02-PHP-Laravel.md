---
title: PHP & Laravel Development
tags: [php, laravel, eloquent, backend]
created: 2026-02-03
---

# PHP & Laravel Development

## 🎯 Core PHP Concepts

### OOP Fundamentals

```php
<?php
// Abstract class
abstract class BaseModel {
    protected $table;
    
    abstract public function validate(): bool;
    
    public function save(): bool {
        if ($this->validate()) {
            // Save logic
            return true;
        }
        return false;
    }
}

// Interface
interface Exportable {
    public function toArray(): array;
    public function toJson(): string;
}

// Trait
trait Timestamps {
    public $created_at;
    public $updated_at;
    
    public function touch(): void {
        $this->updated_at = date('Y-m-d H:i:s');
    }
}

// Implementation
class Student extends BaseModel implements Exportable {
    use Timestamps;
    
    protected $table = 'students';
    public $name;
    public $email;
    
    public function validate(): bool {
        return !empty($this->name) && filter_var($this->email, FILTER_VALIDATE_EMAIL);
    }
    
    public function toArray(): array {
        return ['name' => $this->name, 'email' => $this->email];
    }
    
    public function toJson(): string {
        return json_encode($this->toArray());
    }
}
```

### Error Handling

```php
<?php
// Custom exception
class ValidationException extends Exception {
    protected $errors = [];
    
    public function __construct(array $errors) {
        $this->errors = $errors;
        parent::__construct('Validation failed');
    }
    
    public function getErrors(): array {
        return $this->errors;
    }
}

// Usage
try {
    if (!$student->validate()) {
        throw new ValidationException(['email' => 'Invalid email format']);
    }
    $student->save();
} catch (ValidationException $e) {
    return response()->json(['errors' => $e->getErrors()], 422);
} catch (Exception $e) {
    Log::error($e->getMessage());
    return response()->json(['error' => 'Server error'], 500);
}
```

## 🚀 Laravel Essentials

### Project Structure

```
app/
├── Http/
│   ├── Controllers/
│   │   └── Api/
│   │       ├── StudentController.php
│   │       └── FeeController.php
│   ├── Middleware/
│   └── Requests/
│       └── StoreStudentRequest.php
├── Models/
│   ├── Student.php
│   └── Fee.php
├── Services/
│   └── FeeCalculationService.php
└── Repositories/
    └── StudentRepository.php
```

### Eloquent ORM

```php
<?php
// Model definition
class Student extends Model {
    protected $fillable = ['name', 'email', 'class_id', 'admission_date'];
    
    protected $casts = [
        'admission_date' => 'date',
        'is_active' => 'boolean',
    ];
    
    // Relationships
    public function class(): BelongsTo {
        return $this->belongsTo(ClassRoom::class, 'class_id');
    }
    
    public function fees(): HasMany {
        return $this->hasMany(Fee::class);
    }
    
    public function subjects(): BelongsToMany {
        return $this->belongsToMany(Subject::class)
            ->withPivot('marks', 'grade')
            ->withTimestamps();
    }
    
    // Scopes
    public function scopeActive($query) {
        return $query->where('is_active', true);
    }
    
    public function scopeByClass($query, $classId) {
        return $query->where('class_id', $classId);
    }
    
    // Accessors
    public function getFullNameAttribute(): string {
        return "{$this->first_name} {$this->last_name}";
    }
    
    // Mutators
    public function setEmailAttribute($value): void {
        $this->attributes['email'] = strtolower($value);
    }
}
```

### Eloquent Queries

```php
<?php
// Basic queries
$students = Student::active()->byClass(5)->get();

// Eager loading (N+1 prevention)
$students = Student::with(['class', 'fees' => function($query) {
    $query->where('status', 'pending');
}])->get();

// Aggregations
$totalFees = Fee::where('student_id', $id)
    ->where('status', 'paid')
    ->sum('amount');

// Complex query
$report = Student::select('class_id')
    ->selectRaw('COUNT(*) as total_students')
    ->selectRaw('AVG(fees.amount) as avg_fee')
    ->join('fees', 'students.id', '=', 'fees.student_id')
    ->groupBy('class_id')
    ->having('total_students', '>', 10)
    ->get();
```

### API Resources

```php
<?php
// app/Http/Resources/StudentResource.php
class StudentResource extends JsonResource {
    public function toArray($request): array {
        return [
            'id' => $this->id,
            'name' => $this->full_name,
            'email' => $this->email,
            'class' => new ClassResource($this->whenLoaded('class')),
            'pending_fees' => $this->when(
                $request->user()->isAdmin(),
                $this->fees()->pending()->sum('amount')
            ),
            'created_at' => $this->created_at->toISOString(),
        ];
    }
}

// Usage in controller
public function index() {
    $students = Student::with('class')->paginate(20);
    return StudentResource::collection($students);
}
```

### Form Requests (Validation)

```php
<?php
class StoreStudentRequest extends FormRequest {
    public function authorize(): bool {
        return $this->user()->can('create', Student::class);
    }
    
    public function rules(): array {
        return [
            'name' => 'required|string|max:255',
            'email' => 'required|email|unique:students,email',
            'class_id' => 'required|exists:classes,id',
            'admission_date' => 'required|date|before_or_equal:today',
            'guardian_phone' => 'required|regex:/^[0-9]{10}$/',
        ];
    }
    
    public function messages(): array {
        return [
            'email.unique' => 'This email is already registered.',
            'guardian_phone.regex' => 'Phone must be 10 digits.',
        ];
    }
}
```

### Middleware

```php
<?php
// Custom middleware
class CheckModuleAccess {
    public function handle($request, Closure $next, $module) {
        if (!$request->user()->hasModuleAccess($module)) {
            return response()->json(['error' => 'Access denied'], 403);
        }
        return $next($request);
    }
}

// Registration in Kernel.php
protected $routeMiddleware = [
    'module.access' => \App\Http\Middleware\CheckModuleAccess::class,
];

// Usage
Route::middleware(['auth:api', 'module.access:fees'])
    ->prefix('fees')
    ->group(function () {
        Route::apiResource('invoices', InvoiceController::class);
    });
```

### Service Pattern

```php
<?php
// app/Services/FeeCalculationService.php
class FeeCalculationService {
    public function __construct(
        private FeeStructureRepository $feeRepo,
        private DiscountService $discountService
    ) {}
    
    public function calculateForStudent(Student $student, string $term): array {
        $structure = $this->feeRepo->getForClass($student->class_id, $term);
        $discounts = $this->discountService->getApplicable($student);
        
        $grossAmount = $structure->tuition + $structure->transport + $structure->misc;
        $discountAmount = $this->applyDiscounts($grossAmount, $discounts);
        
        return [
            'gross' => $grossAmount,
            'discount' => $discountAmount,
            'net' => $grossAmount - $discountAmount,
            'breakdown' => $structure->toArray(),
        ];
    }
}

// Controller usage
class FeeController extends Controller {
    public function __construct(private FeeCalculationService $feeService) {}
    
    public function calculate(Student $student, Request $request) {
        $result = $this->feeService->calculateForStudent(
            $student,
            $request->input('term')
        );
        return response()->json($result);
    }
}
```

### Queue Jobs

```php
<?php
// app/Jobs/SendFeeReminder.php
class SendFeeReminder implements ShouldQueue {
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;
    
    public $tries = 3;
    public $backoff = [60, 300, 900]; // Retry delays
    
    public function __construct(public Student $student, public Fee $fee) {}
    
    public function handle(MailService $mailService): void {
        $mailService->sendFeeReminder($this->student, $this->fee);
    }
    
    public function failed(Throwable $exception): void {
        Log::error('Fee reminder failed', [
            'student_id' => $this->student->id,
            'error' => $exception->getMessage()
        ]);
    }
}

// Dispatch
SendFeeReminder::dispatch($student, $fee)->onQueue('emails');
```

## ❓ Interview Q&A

> [!question]- What are Laravel Service Providers?
> Service providers are the central place for bootstrapping Laravel applications. They register bindings in the service container, event listeners, middleware, and routes. Every Laravel application has a `providers` array in `config/app.php`.
> ```php
> class FeeServiceProvider extends ServiceProvider {
>     public function register(): void {
>         $this->app->singleton(FeeCalculationService::class);
>     }
>     
>     public function boot(): void {
>         // Bootstrap services
>     }
> }
> ```

> [!question]- Explain Eloquent relationships
> | Type | Example | Method |
> |------|---------|--------|
> | One-to-One | User → Profile | `hasOne()` / `belongsTo()` |
> | One-to-Many | Class → Students | `hasMany()` / `belongsTo()` |
> | Many-to-Many | Students ↔ Subjects | `belongsToMany()` |
> | Has-Many-Through | School → Students (via Classes) | `hasManyThrough()` |
> | Polymorphic | Comments on Posts/Videos | `morphTo()` / `morphMany()` |

> [!question]- How do you prevent N+1 query problem?
> Use eager loading with `with()`:
> ```php
> // Bad - N+1 queries
> $students = Student::all();
> foreach ($students as $student) {
>     echo $student->class->name; // Query per student
> }
> 
> // Good - 2 queries total
> $students = Student::with('class')->get();
> foreach ($students as $student) {
>     echo $student->class->name; // No additional query
> }
> ```

> [!question]- What is Laravel's Request Lifecycle?
> 1. Request enters `public/index.php`
> 2. Autoloader loaded, application bootstrapped
> 3. Request sent through HTTP Kernel
> 4. Service providers registered and booted
> 5. Request dispatched to router
> 6. Middleware executed (global → route)
> 7. Controller method invoked
> 8. Response returned through middleware
> 9. Response sent to client

> [!question]- How do you implement soft deletes?
> ```php
> // Migration
> $table->softDeletes(); // Adds deleted_at column
> 
> // Model
> use Illuminate\Database\Eloquent\SoftDeletes;
> 
> class Student extends Model {
>     use SoftDeletes;
> }
> 
> // Usage
> $student->delete();           // Soft delete
> Student::withTrashed()->get(); // Include deleted
> Student::onlyTrashed()->get(); // Only deleted
> $student->restore();          // Restore
> $student->forceDelete();      // Permanent delete
> ```

> [!question]- How do you handle database transactions in Laravel?
> ```php
> use Illuminate\Support\Facades\DB;
> 
> // Method 1: Closure (auto commit/rollback)
> DB::transaction(function () use ($student, $fee) {
>     $student->update(['status' => 'enrolled']);
>     Fee::create(['student_id' => $student->id, ...]);
> });
> 
> // Method 2: Manual control
> DB::beginTransaction();
> try {
>     $student->update(['status' => 'enrolled']);
>     Fee::create(['student_id' => $student->id, ...]);
>     DB::commit();
> } catch (Exception $e) {
>     DB::rollBack();
>     throw $e;
> }
> ```

> [!question]- What are Laravel Events and Listeners?
> Events provide a simple observer pattern implementation:
> ```php
> // Event
> class StudentEnrolled {
>     public function __construct(public Student $student) {}
> }
> 
> // Listener
> class SendWelcomeEmail {
>     public function handle(StudentEnrolled $event): void {
>         Mail::to($event->student->email)->send(new WelcomeMail());
>     }
> }
> 
> // Dispatch
> event(new StudentEnrolled($student));
> // Or
> StudentEnrolled::dispatch($student);
> ```

## 🎯 ERP-Specific Laravel Patterns

### Multi-Tenancy (School-wise Data)

```php
<?php
// Global scope for tenant isolation
class TenantScope implements Scope {
    public function apply(Builder $builder, Model $model): void {
        $builder->where('school_id', auth()->user()->school_id);
    }
}

// Model
class Student extends Model {
    protected static function booted(): void {
        static::addGlobalScope(new TenantScope);
        
        static::creating(function ($model) {
            $model->school_id = auth()->user()->school_id;
        });
    }
}
```

### Audit Trail

```php
<?php
trait Auditable {
    protected static function bootAuditable(): void {
        static::created(fn($model) => AuditLog::log('created', $model));
        static::updated(fn($model) => AuditLog::log('updated', $model));
        static::deleted(fn($model) => AuditLog::log('deleted', $model));
    }
}

class Fee extends Model {
    use Auditable;
}
```
