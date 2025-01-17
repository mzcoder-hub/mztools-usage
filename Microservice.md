Here’s an example of how you can integrate the **Core App**, **Cashier App**, and **Accounting App** using REST APIs and message queues.

---

### **Core App (Laravel)**
#### Expose APIs for Authentication and User Management
**routes/api.php**
```php
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Route;

Route::post('/auth/login', [AuthController::class, 'login']);
Route::get('/users', [UserController::class, 'index'])->middleware('auth:sanctum');
Route::post('/users', [UserController::class, 'store']);
```

**AuthController.php**
```php
public function login(Request $request)
{
    $credentials = $request->only('email', 'password');

    if (Auth::attempt($credentials)) {
        $user = Auth::user();
        $token = $user->createToken('auth_token')->plainTextToken;

        return response()->json(['token' => $token], 200);
    }

    return response()->json(['error' => 'Unauthorized'], 401);
}
```

**UserController.php**
```php
public function index()
{
    return response()->json(User::all());
}

public function store(Request $request)
{
    $user = User::create($request->all());
    // Broadcast or send message to other services
    event(new UserCreatedEvent($user));

    return response()->json($user, 201);
}
```

---

### **Cashier App (NestJS)**

#### Fetch User Data from Core App and Process Payment
**UserService.ts**
```typescript
import { HttpService } from '@nestjs/axios';
import { Injectable } from '@nestjs/common';

@Injectable()
export class UserService {
  constructor(private readonly httpService: HttpService) {}

  async getUsers() {
    const response = await this.httpService
      .get('http://core-app-url/api/users', {
        headers: { Authorization: `Bearer YOUR_TOKEN` },
      })
      .toPromise();

    return response.data;
  }
}
```

**PaymentController.ts**
```typescript
import { Controller, Post, Body } from '@nestjs/common';
import { PaymentService } from './payment.service';

@Controller('payments')
export class PaymentController {
  constructor(private readonly paymentService: PaymentService) {}

  @Post()
  async processPayment(@Body() paymentData) {
    const result = await this.paymentService.process(paymentData);
    return { success: true, data: result };
  }
}
```

**PaymentService.ts**
```typescript
import { Injectable } from '@nestjs/common';
import { HttpService } from '@nestjs/axios';
import { EventEmitter2 } from '@nestjs/event-emitter';

@Injectable()
export class PaymentService {
  constructor(
    private readonly httpService: HttpService,
    private readonly eventEmitter: EventEmitter2,
  ) {}

  async process(paymentData) {
    // Process payment here (e.g., Stripe integration)
    const paymentResult = { status: 'success', transactionId: '12345' };

    // Notify Accounting Service
    this.eventEmitter.emit('payment.completed', paymentResult);

    return paymentResult;
  }
}
```

---

### **Accounting App (Laravel)**

#### Listen to Payment Events
**PaymentListener.php**
```php
namespace App\Listeners;

use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Queue\InteractsWithQueue;

class PaymentListener
{
    public function handle($event)
    {
        // Save payment data to the database
        \App\Models\Invoice::create([
            'transaction_id' => $event->transactionId,
            'amount' => $event->amount,
        ]);
    }
}
```

**EventServiceProvider.php**
```php
protected $listen = [
    'payment.completed' => [
        PaymentListener::class,
    ],
];
```

#### Expose APIs for Invoices
**routes/api.php**
```php
Route::get('/invoices', [InvoiceController::class, 'index']);
```

**InvoiceController.php**
```php
public function index()
{
    return response()->json(Invoice::all());
}
```

---

### **Message Queue for Event Communication**
#### Use RabbitMQ for Asynchronous Communication
1. **Core App** publishes events:
   ```php
   use Illuminate\Support\Facades\Queue;

   Queue::push(new \App\Jobs\SendEventJob($eventData));
   ```

2. **Cashier App** listens to the event using a consumer:
   ```typescript
   import { Injectable } from '@nestjs/common';
   import { RabbitMQService } from './rabbitmq.service';

   @Injectable()
   export class EventListener {
     constructor(private readonly rabbitMQService: RabbitMQService) {}

     async listen() {
       this.rabbitMQService.consume('payment.completed', (msg) => {
         console.log('Received:', msg.content.toString());
       });
     }
   }
   ```

3. **Accounting App** processes events:
   ```php
   public function handle($event)
   {
       // Process payment event
   }
   ```

---

This setup ensures smooth communication between your services using REST APIs for synchronous calls and RabbitMQ for asynchronous events.
