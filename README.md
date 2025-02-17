Steps to Implement the Features
1. Set Up Laravel Project
1.	Install Laravel via Composer:
bash
CopyEdit
composer create-project laravel/laravel ride-sharing-portal
cd ride-sharing-portal
2.	Set up a MySQL database and configure the .env file with your database credentials.
________________________________________
2. Authentication
•	Use Laravel's built-in authentication system:
bash
CopyEdit
php artisan make:auth
•	Modify the users table migration:
o	Replace the email column with phone_number and pin:
php
CopyEdit
$table->string('phone_number')->unique();
$table->string('pin', 4); // 4-digit pin
•	Add validation rules for the phone number and pin in the LoginController and registration logic.
Security:
•	Hash the pin using Laravel's Hash facade.
php
CopyEdit
use Illuminate\Support\Facades\Hash;
$hashedPin = Hash::make($request->pin);
•	Use Laravel's middleware to secure routes.
________________________________________
3. Create Models, Migrations, and Seeders
•	Create models and migrations for Customer, Driver, and RideRequest.
bash
CopyEdit
php artisan make:model Customer -m
php artisan make:model Driver -m
php artisan make:model RideRequest -m
•	Define relationships:
o	Customer has many RideRequest.
o	Driver has many RideRequest.
•	Use factories to seed 5 customers, 5 drivers, and some ride requests.
________________________________________
4. Dashboard
•	Create a DashboardController:
bash
CopyEdit
php artisan make:controller DashboardController
•	Add logic to fetch the required counts:
php
CopyEdit
public function index() {
    $customersCount = Customer::count();
    $driversCount = Driver::count();
    $completedRidesCount = RideRequest::where('status', 'completed')->count();
    
    return view('dashboard', compact('customersCount', 'driversCount', 'completedRidesCount'));
}
•	Create a dashboard.blade.php view to display the summary.
________________________________________
5. Customer List with Filters
•	Create a CustomerController:
bash
CopyEdit
php artisan make:controller CustomerController
•	Add logic for fetching customers:
php
CopyEdit
public function index(Request $request) {
    $customers = Customer::orderBy('created_at', 'desc');

    if ($request->has('registration_date')) {
        $customers->whereDate('created_at', $request->registration_date);
    }

    return view('customers.index', ['customers' => $customers->get()]);
}
•	Create a customers/index.blade.php view with a filter form.
________________________________________
6. Driver List with Filters
•	Create a DriverController:
bash
CopyEdit
php artisan make:controller DriverController
•	Add filtering logic:
php
CopyEdit
public function index(Request $request) {
    $drivers = Driver::query();

    if ($request->has('county')) {
        $drivers->where('county', $request->county);
    }

    if ($request->has('sub_county')) {
        $drivers->where('sub_county', $request->sub_county);
    }

    return view('drivers.index', ['drivers' => $drivers->orderBy('created_at', 'desc')->get()]);
}
•	Create a drivers/index.blade.php view with dropdowns for counties and sub-counties.
________________________________________
7. Ride Request List
•	Create a RideRequestController:
bash
CopyEdit
php artisan make:controller RideRequestController
•	Add logic for fetching ride requests:
php
CopyEdit
public function index() {
    $rideRequests = RideRequest::orderBy('created_at', 'desc')->get();
    return view('ride_requests.index', ['rideRequests' => $rideRequests]);
}
•	Create a ride_requests/index.blade.php view.
________________________________________
8. Routes
Update the web.php file:
php
CopyEdit
use App\Http\Controllers\DashboardController;
use App\Http\Controllers\CustomerController;
use App\Http\Controllers\DriverController;
use App\Http\Controllers\RideRequestController;

Route::get('/', [DashboardController::class, 'index'])->middleware('auth');

Route::resource('customers', CustomerController::class)->middleware('auth');
Route::resource('drivers', DriverController::class)->middleware('auth');
Route::resource('ride-requests', RideRequestController::class)->middleware('auth');
________________________________________
9. UI Enhancements
•	Use Laravel Blade templates for consistent layout.
•	Consider using Bootstrap or TailwindCSS for styling.
•	Add pagination for lists using ->paginate().
