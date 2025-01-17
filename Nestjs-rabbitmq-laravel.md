To connect **NestJS** (Cashier App) to **RabbitMQ** for event communication and integrate it with the **Accounting App**, you can use the `@nestjs/microservices` package. Here's how you can set it up:

---

### **NestJS (Cashier App): Event Emitter and RabbitMQ Integration**

#### 1. Install RabbitMQ Dependencies
Install the required packages for RabbitMQ:
```bash
npm install @nestjs/microservices amqplib
```

#### 2. Configure RabbitMQ in NestJS
Create a RabbitMQ service and configure it.

**rabbitmq.module.ts**
```typescript
import { Module } from '@nestjs/common';
import { ClientsModule, Transport } from '@nestjs/microservices';

@Module({
  imports: [
    ClientsModule.register([
      {
        name: 'RABBITMQ_SERVICE',
        transport: Transport.RMQ,
        options: {
          urls: ['amqp://localhost:5672'], // Replace with your RabbitMQ URL
          queue: 'payment_queue',
          queueOptions: {
            durable: true,
          },
        },
      },
    ]),
  ],
  exports: [ClientsModule],
})
export class RabbitMQModule {}
```

#### 3. Emit Events to RabbitMQ
Use the `ClientProxy` to publish events to RabbitMQ.

**payment.service.ts**
```typescript
import { Injectable } from '@nestjs/common';
import { ClientProxy } from '@nestjs/microservices';
import { Inject } from '@nestjs/common';

@Injectable()
export class PaymentService {
  constructor(
    @Inject('RABBITMQ_SERVICE') private readonly rabbitMQClient: ClientProxy,
  ) {}

  async processPayment(paymentData: any) {
    // Process payment logic here (e.g., integrating with Stripe)

    // Emit event to RabbitMQ
    const paymentEvent = {
      transactionId: '12345',
      amount: paymentData.amount,
      userId: paymentData.userId,
      status: 'success',
    };

    this.rabbitMQClient.emit('payment.completed', paymentEvent);
    return paymentEvent;
  }
}
```

#### 4. Trigger Event in Controller
**payment.controller.ts**
```typescript
import { Controller, Post, Body } from '@nestjs/common';
import { PaymentService } from './payment.service';

@Controller('payments')
export class PaymentController {
  constructor(private readonly paymentService: PaymentService) {}

  @Post()
  async processPayment(@Body() paymentData: any) {
    const result = await this.paymentService.processPayment(paymentData);
    return { success: true, result };
  }
}
```

---

### **Laravel (Accounting App): Consuming Events from RabbitMQ**

#### 1. Install RabbitMQ Laravel Package
Install a RabbitMQ library like `php-amqplib` or `beyondcode/laravel-queue-rabbitmq`.

```bash
composer require vyuldashev/laravel-queue-rabbitmq
```

#### 2. Configure RabbitMQ
Update the `config/queue.php` file to include RabbitMQ as a driver.

**config/queue.php**
```php
'rabbitmq' => [
    'driver' => 'rabbitmq',
    'queue' => env('RABBITMQ_QUEUE', 'payment_queue'),
    'connection' => PhpAmqpLib\Connection\AMQPStreamConnection::class,
    'hosts' => [
        [
            'host' => env('RABBITMQ_HOST', '127.0.0.1'),
            'port' => env('RABBITMQ_PORT', 5672),
            'user' => env('RABBITMQ_USER', 'guest'),
            'password' => env('RABBITMQ_PASSWORD', 'guest'),
            'vhost' => env('RABBITMQ_VHOST', '/'),
        ],
    ],
    'options' => [
        'ssl_options' => [
            'verify_peer' => false,
            'verify_peer_name' => false,
        ],
    ],
],
```

#### 3. Set Up a Listener for RabbitMQ Events
Create a job to handle the `payment.completed` event.

**app/Jobs/ProcessPaymentEvent.php**
```php
namespace App\Jobs;

use Illuminate\Contracts\Queue\ShouldQueue;

class ProcessPaymentEvent implements ShouldQueue
{
    public function __construct(public $paymentEvent) {}

    public function handle()
    {
        // Save payment data to the database
        \App\Models\Invoice::create([
            'transaction_id' => $this->paymentEvent['transactionId'],
            'amount' => $this->paymentEvent['amount'],
            'user_id' => $this->paymentEvent['userId'],
            'status' => $this->paymentEvent['status'],
        ]);
    }
}
```

#### 4. Listen for Events in Queue Worker
Queue workers will consume RabbitMQ messages automatically.

**app/Providers/EventServiceProvider.php**
```php
use App\Jobs\ProcessPaymentEvent;

protected $listen = [
    'payment.completed' => [
        ProcessPaymentEvent::class,
    ],
];
```

#### 5. Start Queue Worker
Run the Laravel queue worker to process events.

```bash
php artisan queue:work --queue=rabbitmq
```

---

### **Summary Workflow**
1. **Cashier App (NestJS)**:
   - Processes a payment.
   - Emits a `payment.completed` event to RabbitMQ.

2. **RabbitMQ**:
   - Acts as the message broker, delivering the event to the Accounting App's queue.

3. **Accounting App (Laravel)**:
   - Consumes the `payment.completed` event.
   - Saves the payment data into the database for invoicing.

This approach decouples services and ensures scalability with RabbitMQ handling the communication.
