# TMMMS 
## Truck Monitoring Management System

Detailed design for your database with tables and relationships for the truck tracking application in Laravel:

### Tables and Relationships

1. **Users Table**
   - Stores information about all users, including companies, customers, depots, unions, and admins.
   - Relationships:
     - One-to-Many with Trucks
     - One-to-Many with Locations
     - One-to-Many with Messages
     - One-to-Many with Transactions

   ```sql
   CREATE TABLE users (
       id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
       name VARCHAR(255),
       email VARCHAR(255) UNIQUE,
       password VARCHAR(255),
       phone_number VARCHAR(20),
       role ENUM('company', 'customer', 'depot', 'union', 'admin'),
       created_at TIMESTAMP,
       updated_at TIMESTAMP
   );
   ```

2. **Trucks Table**
   - Stores information about trucks.
   - Relationships:
     - Belongs to a User (Company)
     - One-to-Many with Locations
     - One-to-Many with Programs
     - One-to-Many with Disputes

   ```sql
   CREATE TABLE trucks (
       id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
       user_id BIGINT UNSIGNED,
       truck_number VARCHAR(50),
       calibration VARCHAR(50),
       created_at TIMESTAMP,
       updated_at TIMESTAMP,
       FOREIGN KEY (user_id) REFERENCES users(id)
   );
   ```

3. **Locations Table**
   - Stores real-time location updates for trucks.
   - Relationships:
     - Belongs to a Truck
     - Belongs to a User (Driver)

   ```sql
   CREATE TABLE locations (
       id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
       truck_id BIGINT UNSIGNED,
       user_id BIGINT UNSIGNED,
       latitude DECIMAL(10, 8),
       longitude DECIMAL(11, 8),
       timestamp TIMESTAMP,
       created_at TIMESTAMP,
       updated_at TIMESTAMP,
       FOREIGN KEY (truck_id) REFERENCES trucks(id),
       FOREIGN KEY (user_id) REFERENCES users(id)
   );
   ```

4. **Programs Table**
   - Stores information about transportation programs.
   - Relationships:
     - Belongs to a Truck

   ```sql
   CREATE TABLE programs (
       id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
       truck_id BIGINT UNSIGNED,
       driver_name VARCHAR(255),
       product_type VARCHAR(255),
       quantity INT,
       eta TIMESTAMP,
       waybill_number VARCHAR(50),
       created_at TIMESTAMP,
       updated_at TIMESTAMP,
       FOREIGN KEY (truck_id) REFERENCES trucks(id)
   );
   ```

5. **Disputes Table**
   - Stores information about disputes.
   - Relationships:
     - Belongs to a Truck

   ```sql
   CREATE TABLE disputes (
       id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
       truck_id BIGINT UNSIGNED,
       dispute_type VARCHAR(255),
       description TEXT,
       created_at TIMESTAMP,
       updated_at TIMESTAMP,
       FOREIGN KEY (truck_id) REFERENCES trucks(id)
   );
   ```

6. **Messages Table**
   - Stores messages and call logs.
   - Relationships:
     - Belongs to a User

   ```sql
   CREATE TABLE messages (
       id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
       user_id BIGINT UNSIGNED,
       content TEXT,
       message_type ENUM('text', 'call'),
       created_at TIMESTAMP,
       updated_at TIMESTAMP,
       FOREIGN KEY (user_id) REFERENCES users(id)
   );
   ```

7. **Transactions Table**
   - Stores information about transactions made through the virtual wallet.
   - Relationships:
     - Belongs to a User

   ```sql
   CREATE TABLE transactions (
       id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
       user_id BIGINT UNSIGNED,
       amount DECIMAL(10, 2),
       transaction_type ENUM('deposit', 'withdrawal'),
       created_at TIMESTAMP,
       updated_at TIMESTAMP,
       FOREIGN KEY (user_id) REFERENCES users(id)
   );
   ```

### Relationships Overview
- **Users** can have multiple **Trucks**, **Locations**, **Messages**, and **Transactions**.
- **Trucks** belong to **Users** (Companies) and can have multiple **Locations**, **Programs**, and **Disputes**.
- **Locations** belong to both **Trucks** and **Users** (Drivers).
- **Programs** and **Disputes** belong to **Trucks**.

### Eloquent Models and Relationships in Laravel

1. **User Model**
   ```php
   class User extends Authenticatable
   {
       public function trucks()
       {
           return $this->hasMany(Truck::class);
       }

       public function locations()
       {
           return $this->hasMany(Location::class);
       }

       public function messages()
       {
           return $this->hasMany(Message::class);
       }

       public function transactions()
       {
           return $this->hasMany(Transaction::class);
       }
   }
   ```

2. **Truck Model**
   ```php
   class Truck extends Model
   {
       public function user()
       {
           return $this->belongsTo(User::class);
       }

       public function locations()
       {
           return $this->hasMany(Location::class);
       }

       public function programs()
       {
           return $this->hasMany(Program::class);
       }

       public function disputes()
       {
           return $this->hasMany(Dispute::class);
       }
   }
   ```

3. **Location Model**
   ```php
   class Location extends Model
   {
       public function truck()
       {
           return $this->belongsTo(Truck::class);
       }

       public function user()
       {
           return $this->belongsTo(User::class);
       }
   }
   ```

4. **Program Model**
   ```php
   class Program extends Model
   {
       public function truck()
       {
           return $this->belongsTo(Truck::class);
       }
   }
   ```

5. **Dispute Model**
   ```php
   class Dispute extends Model
   {
       public function truck()
       {
           return $this->belongsTo(Truck::class);
       }
   }
   ```

6. **Message Model**
   ```php
   class Message extends Model
   {
       public function user()
       {
           return $this->belongsTo(User::class);
       }
   }
   ```

7. **Transaction Model**
   ```php
   class Transaction extends Model
   {
       public function user()
       {
           return $this->belongsTo(User::class);
       }
   }
   ```

This database design and the corresponding Eloquent relationships should provide a solid foundation for building your truck tracking web application in Laravel.







Continuing with the detailed explanation for building your truck tracking web application:

### Additional Features and Considerations

#### Real-time Location Tracking
- **WebSockets**: Use Laravel Echo and Pusher for real-time updates.
  - Install Laravel Echo and Pusher.
    ```bash
    composer require pusher/pusher-php-server
    npm install --save laravel-echo pusher-js
    ```
  - Set up broadcasting configuration in `.env`:
    ```env
    BROADCAST_DRIVER=pusher
    PUSHER_APP_ID=your-app-id
    PUSHER_APP_KEY=your-app-key
    PUSHER_APP_SECRET=your-app-secret
    PUSHER_APP_CLUSTER=your-app-cluster
    ```
  - Configure broadcasting in `config/broadcasting.php` and `config/queue.php`.

- **JavaScript**: Implement the front-end to listen for real-time updates.
  ```javascript
  Echo.channel('truck-locations')
      .listen('LocationUpdated', (e) => {
          // Update map with new location
      });
  ```

#### Secure API Endpoints
- **API Routes**: Create secure routes for the API.
  - Define routes in `routes/api.php`.
  ```php
  Route::middleware('auth:sanctum')->group(function () {
      Route::get('/trucks', [TruckController::class, 'index']);
      Route::post('/locations', [LocationController::class, 'store']);
      // Add more routes as needed
  });
  ```

- **API Controllers**: Implement controllers for handling API requests.
  ```php
  class TruckController extends Controller
  {
      public function index()
      {
          $trucks = Truck::with('locations')->get();
          return response()->json($trucks);
      }
  }
  ```

#### Front-end Development
- **Blade Templates**: Utilize Blade templating for dynamic front-end rendering.
- **Vue.js or React**: For a more interactive front-end, consider integrating Vue.js or React.
  - Install Vue.js.
    ```bash
    npm install vue
    ```

- **Admin LTE**: Use Admin LTE for an admin dashboard template.
  - Download Admin LTE and integrate it into your Blade templates.
    ```html
    <!DOCTYPE html>
    <html lang="en">
    <head>
        @include('layouts.partials.head')
    </head>
    <body class="hold-transition sidebar-mini">
        <div class="wrapper">
            @include('layouts.partials.navbar')
            @include('layouts.partials.sidebar')
            <div class="content-wrapper">
                @yield('content')
            </div>
            @include('layouts.partials.footer')
        </div>
        @include('layouts.partials.scripts')
    </body>
    </html>
    ```

#### Deployment and Maintenance
- **Continuous Integration (CI)**: Use GitHub Actions to set up CI for automatic testing and deployment.
  - Example GitHub Actions workflow:
    ```yaml
    name: Laravel CI

    on: [push]

    jobs:
      test:
        runs-on: ubuntu-latest

        steps:
          - uses: actions/checkout@v2
          - name: Set up PHP
            uses: shivammathur/setup-php@v2
            with:
              php-version: '7.4'
          - name: Install dependencies
            run: composer install
          - name: Run tests
            run: php artisan test
    ```

- **Hosting**: Choose a reliable hosting provider (e.g., AWS, DigitalOcean, or Heroku).
- **SSL Certificates**: Ensure the application uses HTTPS by obtaining and installing SSL certificates.

### Summary

This comprehensive guide outlines the steps to develop a Laravel-based truck tracking web application. By following these steps, you can build a robust, scalable, and secure application that meets the needs of your users, including companies, customers, depots, unions, and administrators. The key features include real-time location tracking, role-based access control, virtual wallet integration, and comprehensive data management.

#### Key Points:
1. **Set Up**: Laravel installation and environment configuration.
2. **Authentication**: User authentication using Laravel Breeze or Jetstream.
3. **Database Design**: Creating tables and relationships.
4. **Feature Implementation**: Real-time tracking, dashboards, messaging, and role-based access control.
5. **Front-end Development**: Using Blade, Vue.js/React, and Admin LTE.
6. **API Development**: Secure API endpoints and controllers.
7. **Deployment**: CI/CD pipeline setup and secure hosting.

By implementing these features and following best practices, you will be able to create an efficient and user-friendly truck tracking application that provides real-time insights and improves operational efficiency.





Sure! Below are the migration files for each of the database tables described. 

### Migration for `users` Table
```php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreateUsersTable extends Migration
{
    public function up()
    {
        Schema::create('users', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->string('email')->unique();
            $table->string('password');
            $table->string('phone_number')->nullable();
            $table->enum('role', ['company', 'customer', 'depot', 'union', 'admin']);
            $table->timestamps();
        });
    }

    public function down()
    {
        Schema::dropIfExists('users');
    }
}
```

### Migration for `trucks` Table
```php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreateTrucksTable extends Migration
{
    public function up()
    {
        Schema::create('trucks', function (Blueprint $table) {
            $table->id();
            $table->foreignId('user_id')->constrained()->onDelete('cascade');
            $table->string('truck_number');
            $table->string('calibration');
            $table->timestamps();
        });
    }

    public function down()
    {
        Schema::dropIfExists('trucks');
    }
}
```

### Migration for `locations` Table
```php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreateLocationsTable extends Migration
{
    public function up()
    {
        Schema::create('locations', function (Blueprint $table) {
            $table->id();
            $table->foreignId('truck_id')->constrained()->onDelete('cascade');
            $table->foreignId('user_id')->constrained()->onDelete('cascade');
            $table->decimal('latitude', 10, 8);
            $table->decimal('longitude', 11, 8);
            $table->timestamp('timestamp')->useCurrent();
            $table->timestamps();
        });
    }

    public function down()
    {
        Schema::dropIfExists('locations');
    }
}
```

### Migration for `programs` Table
```php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreateProgramsTable extends Migration
{
    public function up()
    {
        Schema::create('programs', function (Blueprint $table) {
            $table->id();
            $table->foreignId('truck_id')->constrained()->onDelete('cascade');
            $table->string('driver_name');
            $table->string('product_type');
            $table->integer('quantity');
            $table->timestamp('eta')->useCurrent();
            $table->string('waybill_number');
            $table->timestamps();
        });
    }

    public function down()
    {
        Schema::dropIfExists('programs');
    }
}
```

### Migration for `disputes` Table
```php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreateDisputesTable extends Migration
{
    public function up()
    {
        Schema::create('disputes', function (Blueprint $table) {
            $table->id();
            $table->foreignId('truck_id')->constrained()->onDelete('cascade');
            $table->string('dispute_type');
            $table->text('description');
            $table->timestamps();
        });
    }

    public function down()
    {
        Schema::dropIfExists('disputes');
    }
}
```

### Migration for `messages` Table
```php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreateMessagesTable extends Migration
{
    public function up()
    {
        Schema::create('messages', function (Blueprint $table) {
            $table->id();
            $table->foreignId('user_id')->constrained()->onDelete('cascade');
            $table->text('content');
            $table->enum('message_type', ['text', 'call']);
            $table->timestamps();
        });
    }

    public function down()
    {
        Schema::dropIfExists('messages');
    }
}
```

### Migration for `transactions` Table
```php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreateTransactionsTable extends Migration
{
    public function up()
    {
        Schema::create('transactions', function (Blueprint $table) {
            $table->id();
            $table->foreignId('user_id')->constrained()->onDelete('cascade');
            $table->decimal('amount', 10, 2);
            $table->enum('transaction_type', ['deposit', 'withdrawal']);
            $table->timestamps();
        });
    }

    public function down()
    {
        Schema::dropIfExists('transactions');
    }
}
```

### Laravel Command to Create Migration Files
You can create these migration files using the Laravel Artisan command. For example:

```bash
php artisan make:migration create_users_table
php artisan make:migration create_trucks_table
php artisan make:migration create_locations_table
php artisan make:migration create_programs_table
php artisan make:migration create_disputes_table
php artisan make:migration create_messages_table
php artisan make:migration create_transactions_table
```

After creating the migration files, you can copy the respective content provided above into the created files.

Finally, run the migration to create the tables in the database:

```bash
php artisan migrate
```

This will create the database tables with the specified relationships and constraints.