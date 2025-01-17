If the **Accounting App** is Laravel and needs to fetch user details from the **Core App** while maintaining its own database for independent operations, the setup can be implemented as follows:

---

### **Architecture Overview**

1. **Core App (Laravel)**:
   - Provides authentication and user management APIs.
   - Issues tokens (e.g., JWT or Sanctum) for secure communication.

2. **Accounting App (Laravel)**:
   - Authenticates requests by validating tokens with the Core App.
   - Fetches user details from the Core App via REST APIs when needed.
   - Manages accounting operations (e.g., invoices, reports) in its own database.

---

### **Implementation Steps**

#### **1. Core App: Expose Login and User APIs**
Already implemented as shown in the previous example. Ensure these endpoints are protected with authentication middleware (e.g., Sanctum).

---

#### **2. Accounting App: Integrate with Core App**

##### **a. Add Environment Variables**
Add the Core App URL and credentials to the `.env` file in the Accounting App:
```env
CORE_APP_URL=http://core-app-url/api
CORE_APP_TOKEN=your_static_service_token
```

---

##### **b. Create a Service for Core App Communication**
Create a service to handle communication with the Core App.

**app/Services/CoreAppService.php**
```php
namespace App\Services;

use Illuminate\Support\Facades\Http;

class CoreAppService
{
    protected $coreAppUrl;
    protected $coreAppToken;

    public function __construct()
    {
        $this->coreAppUrl = config('services.core_app.url');
        $this->coreAppToken = config('services.core_app.token');
    }

    public function getUserDetails($userId)
    {
        $response = Http::withToken($this->coreAppToken)
            ->get("{$this->coreAppUrl}/users/{$userId}");

        if ($response->successful()) {
            return $response->json();
        }

        return null;
    }
}
```

---

##### **c. Configure Core App Service**
Update the `config/services.php` file to include Core App configurations:
```php
return [
    'core_app' => [
        'url' => env('CORE_APP_URL'),
        'token' => env('CORE_APP_TOKEN'),
    ],
];
```

---

##### **d. Use CoreAppService in Controllers**
Use the service to fetch user details and handle accounting operations.

**app/Http/Controllers/InvoiceController.php**
```php
namespace App\Http\Controllers;

use App\Models\Invoice;
use App\Services\CoreAppService;
use Illuminate\Http\Request;

class InvoiceController extends Controller
{
    protected $coreAppService;

    public function __construct(CoreAppService $coreAppService)
    {
        $this->coreAppService = $coreAppService;
    }

    public function create(Request $request)
    {
        // Fetch user details from Core App
        $user = $this->coreAppService->getUserDetails($request->user_id);

        if (!$user) {
            return response()->json(['error' => 'User not found'], 404);
        }

        // Create an invoice in the Accounting App database
        $invoice = Invoice::create([
            'user_id' => $user['id'],
            'amount' => $request->amount,
            'status' => 'pending',
        ]);

        return response()->json($invoice, 201);
    }

    public function index()
    {
        // Fetch all invoices
        $invoices = Invoice::all();
        return response()->json($invoices);
    }
}
```

---

##### **e. Invoice Migration and Model**
Create the `invoices` table and corresponding model.

**database/migrations/xxxx_xx_xx_create_invoices_table.php**
```php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreateInvoicesTable extends Migration
{
    public function up()
    {
        Schema::create('invoices', function (Blueprint $table) {
            $table->id();
            $table->unsignedBigInteger('user_id');
            $table->decimal('amount', 10, 2);
            $table->string('status');
            $table->timestamps();
        });
    }

    public function down()
    {
        Schema::dropIfExists('invoices');
    }
}
```

**app/Models/Invoice.php**
```php
namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Invoice extends Model
{
    use HasFactory;

    protected $fillable = ['user_id', 'amount', 'status'];
}
```

---

##### **f. Routes**
Add routes for invoice management in **routes/api.php**:
```php
use App\Http\Controllers\InvoiceController;

Route::middleware('auth:sanctum')->group(function () {
    Route::post('/invoices', [InvoiceController::class, 'create']);
    Route::get('/invoices', [InvoiceController::class, 'index']);
});
```

---

### **Example Workflow**

1. **User Login**:
   - The Cashier App or Accounting App authenticates users via the Core App's `/auth/login` endpoint.
   - The token received is stored and used for subsequent API calls.

2. **Create Invoice**:
   - The Accounting App receives a request to create an invoice.
   - It fetches user details from the Core App using the `CoreAppService`.
   - The invoice is stored in the Accounting App's database.

3. **Fetch Invoices**:
   - The Accounting App retrieves and serves invoices from its database.

---

### **Key Points**
1. **Core App**:
   - Acts as the source of truth for authentication and user data.
   - Exposes APIs to fetch user details securely.

2. **Accounting App**:
   - Manages its own database for accounting operations (e.g., invoices, reports).
   - Fetches user details from the Core App only when necessary.

3. **Separation of Responsibility**:
   - Core App: User management and authentication.
   - Accounting App: Financial data and operations.

4. **Loose Coupling**:
   - The Accounting App remains independent, relying on the Core App only for shared user details.

This setup ensures each service is independently scalable and secure while maintaining necessary integration points.
