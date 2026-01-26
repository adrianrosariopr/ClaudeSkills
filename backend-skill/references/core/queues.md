<overview>
Background job processing and message queues. Essential for offloading slow operations, handling spikes, and building reliable distributed systems.
</overview>

<when_to_queue>
**Queue These Operations:**

- Sending emails/SMS
- Processing uploads (images, videos)
- Generating reports/PDFs
- Calling external APIs
- Data imports/exports
- Webhook deliveries
- Search index updates
- Notifications (push, in-app)
- Heavy computations
- Anything taking > 100ms

**Don't Queue:**

- User authentication
- Simple CRUD operations
- Real-time data fetching
- Anything user is waiting for
</when_to_queue>

<job_patterns>
**Basic Job Structure:**

```php
// Laravel
class SendWelcomeEmail implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    public function __construct(
        public User $user
    ) {}

    public function handle(): void
    {
        Mail::to($this->user)->send(new WelcomeEmail($this->user));
    }

    public function failed(\Throwable $exception): void
    {
        Log::error('Welcome email failed', [
            'user_id' => $this->user->id,
            'error' => $exception->getMessage(),
        ]);
    }
}

// Dispatch
SendWelcomeEmail::dispatch($user);
SendWelcomeEmail::dispatch($user)->delay(now()->addMinutes(5));
SendWelcomeEmail::dispatch($user)->onQueue('emails');
```

**Node.js (BullMQ):**

```typescript
import { Queue, Worker } from 'bullmq';

const emailQueue = new Queue('email');

// Add job
await emailQueue.add('welcome', { userId: 123 });

// Process jobs
const worker = new Worker('email', async (job) => {
  const user = await User.findById(job.data.userId);
  await sendEmail(user);
});
```
</job_patterns>

<retry_strategies>
**Retry Configuration:**

```php
// Laravel
class ProcessPayment implements ShouldQueue
{
    public $tries = 3;
    public $backoff = [30, 60, 120];  // Exponential backoff
    public $maxExceptions = 2;

    // Or calculate dynamically
    public function retryUntil(): DateTime
    {
        return now()->addHours(24);
    }
}
```

**Retry Patterns:**

```
Attempt 1: Immediate
Attempt 2: Wait 30s
Attempt 3: Wait 60s
Attempt 4: Wait 120s
...
Move to dead letter queue
```

**Idempotency:**

```php
class ProcessPayment implements ShouldQueue
{
    public function handle(): void
    {
        // Use unique key to prevent duplicate processing
        $lockKey = "payment:{$this->order->id}";

        if (!Cache::lock($lockKey, 300)->get()) {
            return;  // Already processing
        }

        try {
            // Check if already processed
            if ($this->order->paid_at) {
                return;
            }

            $this->processPayment();
            $this->order->update(['paid_at' => now()]);
        } finally {
            Cache::lock($lockKey)->release();
        }
    }
}
```
</retry_strategies>

<queue_configuration>
**Queue Drivers:**

```php
// Redis (recommended for most cases)
QUEUE_CONNECTION=redis

// Database (simple, no extra infrastructure)
QUEUE_CONNECTION=database

// Amazon SQS (managed, scalable)
QUEUE_CONNECTION=sqs

// RabbitMQ (advanced routing)
QUEUE_CONNECTION=rabbitmq
```

**Multiple Queues:**

```php
// Separate by priority/type
dispatch(new SendEmail($user))->onQueue('emails');
dispatch(new ProcessVideo($video))->onQueue('media');
dispatch(new ImportData($file))->onQueue('imports');

// Run workers per queue
php artisan queue:work --queue=emails
php artisan queue:work --queue=media --timeout=3600
php artisan queue:work --queue=imports
```

**Priority Queues:**

```bash
# Process high priority first
php artisan queue:work --queue=high,default,low
```
</queue_configuration>

<job_batching>
**Batch Processing:**

```php
// Laravel Batch
$batch = Bus::batch([
    new ProcessPodcast($podcast1),
    new ProcessPodcast($podcast2),
    new ProcessPodcast($podcast3),
])->then(function (Batch $batch) {
    // All jobs completed
})->catch(function (Batch $batch, \Throwable $e) {
    // First failure
})->finally(function (Batch $batch) {
    // Batch finished (success or fail)
})->dispatch();

// Check progress
$batch = Bus::findBatch($batchId);
$batch->progress();  // 0-100
$batch->finished();  // bool
```

**Job Chaining:**

```php
// Execute in sequence
Bus::chain([
    new ProcessUpload($file),
    new GenerateThumbnails($file),
    new UpdateSearchIndex($file),
    new NotifyUser($user),
])->dispatch();
```
</job_batching>

<monitoring>
**Queue Monitoring:**

```php
// Laravel Horizon (Redis)
// Dashboard at /horizon
// Monitor:
// - Job throughput
// - Wait times
// - Failed jobs
// - Worker status

// Metrics to track
- Queue length (jobs waiting)
- Processing time (per job type)
- Failure rate
- Worker utilization
```

**Alerting Thresholds:**

```yaml
alerts:
  - queue_length > 1000: warning
  - queue_length > 5000: critical
  - oldest_job > 5m: warning
  - failure_rate > 5%: warning
```
</monitoring>

<failure_handling>
**Dead Letter Queues:**

```php
// Jobs that exceed retries go to failed_jobs table
// Review and retry
php artisan queue:failed
php artisan queue:retry all
php artisan queue:retry 5  // Specific job

// Programmatic retry
$failedJob = DB::table('failed_jobs')->find($id);
Artisan::call('queue:retry', ['id' => $id]);
```

**Graceful Shutdown:**

```php
// Worker options
php artisan queue:work --stop-when-empty
php artisan queue:restart  // Signal all workers to restart

// In job
public function handle(): void
{
    // Long running job
    foreach ($items as $item) {
        $this->process($item);

        // Check if shutdown requested
        if (app('queue.worker')->shouldQuit) {
            $this->release(60);  // Re-queue for later
            return;
        }
    }
}
```
</failure_handling>

<scaling>
**Scaling Workers:**

```bash
# Supervisor config
[program:queue-worker]
command=php artisan queue:work --queue=default --tries=3
numprocs=4
autostart=true
autorestart=true
```

**Horizontal Scaling:**

```
- Add more worker processes
- Separate queues by job type
- Use managed queue services (SQS, Cloud Tasks)
- Consider serverless (Lambda, Cloud Functions)
```

**Rate Limiting:**

```php
// Laravel
Redis::throttle('emails')
    ->allow(100)
    ->every(60)
    ->then(function () {
        // Send email
    }, function () {
        // Rate limited, re-queue
        return $this->release(60);
    });
```
</scaling>

<decision_tree>
**Queue Driver Selection:**

Use Redis when:
- Speed is critical
- Need job prioritization
- Want Horizon dashboard
- Moderate scale

Use Database when:
- Simple requirements
- No Redis available
- Low job volume

Use SQS when:
- High scale
- Managed infrastructure
- AWS ecosystem

Use RabbitMQ when:
- Complex routing
- Multiple consumers
- Message acknowledgment patterns
</decision_tree>
