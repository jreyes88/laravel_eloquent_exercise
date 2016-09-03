# laravel_eloquent_exercise

Eloquent is an ORM for laravel.

We will use Eloquent to do queries with SQL

// 1. Create project with .env
composer create-project laravel/laravel tempProject 5.2.x

// 2. go into project
cd tempProject

// 3. Laravel will create the login/register stuff
php artisan make:auth

// 4. show hidden files
ls -a

// 5. open the environment
open .env

// 6. adjust the database connection:
APP_ENV=local
APP_DEBUG=true
APP_KEY=base64:uOoDUiNa7yjp8MG0WAzZQbaFCnG/RdssYKFupOCczY0=
APP_URL=http://localhost

DB_CONNECTION=mysql
DB_HOST=localhost
DB_PORT=3306
DB_DATABASE=[dbname]
DB_USERNAME=root
DB_PASSWORD=

CACHE_DRIVER=file
SESSION_DRIVER=file
QUEUE_DRIVER=sync

REDIS_HOST=127.0.0.1
REDIS_PASSWORD=null
REDIS_PORT=6379

MAIL_DRIVER=smtp
MAIL_HOST=mailtrap.io
MAIL_PORT=2525
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null


// 7. IN SQL:

CREATE DATABASE [dbname];

USE [dbname];
CREATE TABLE users
(
    id int NOT NULL AUTO_INCREMENT,
    name varchar(255) NOT NULL,
    email varchar(255) NOT NULL,
    password varchar(255) NOT NULL,
    updated_at varchar(255) NOT NULL,
    created_at varchar(255) NOT NULL,
    remember_token varchar(255) NOT NULL,
    PRIMARY KEY (id)
)

// 8. this will show route
php artisan route:list

// 9. go to app>>Http>>routes.php and add:
Route::resource('photo', 'PhotoController');

// 10. in terminal:
php artisan make:controller PhotoController --resource

// 11. in terminal:
php artisan make:seeder UserTableSeeder

//12. in database>>UserTableSeeder.php add this to line 5:
use App\User;

// 13. then in line 16:
// This will create 50 users
factory(User::class, 50)->create();

// 14. go into DatabseSeeder.php
un-comment line 14 but make sure it's
$this->call(UserTableSeeder::class);

// 15. to RUN the seeder and make those 50 users:
php artisan db:seed

// 16. in terminal
//gives you shell and you can certain commands to make sure things are working
php artisan tinker
App\User::find(1)

// 17. CTRL C to get out of Tinker environment

// 18. create users controller
php artisan make:controller UserController --resource

// 19. In app>>Http>>routes.php line 24 add:
Route::resource('users', 'UserController');

// 20. app>>Http>>UserController.php line 51 (in public function show($id)):
$user = User::find($id);
var_export($user);

// 21. SAME FILE ^ line 9
use App\User;

// 22. now we start trying to show the user info on the page
IN THE SAME FILE line 56:
return view('user', $user);

// 22. create user.blade.php:
@extends('layouts.app')

@section('content')
<div class="container">
    <div class="row">
        <div class="col-md-10 col-md-offset-1">
            <div class="panel panel-default">
                <div class="panel-heading">User Info</div>

                <div class="panel-body">
                    <p> {{ $name }}</p>
                </div>
            </div>
        </div>
    </div>
</div>
@endsection


// 23. to display all users
// In usercontroller.php
// in public function index():

$users = User::all();

return view('users', compact('users'));

// 24. create users.blade.php:
@extends('layouts.app')

@section('content')
<div class="container">
    <div class="row">
        <div class="col-md-10 col-md-offset-1">
            <div class="panel panel-default">
                <div class="panel-heading">All Users</div>

                <div class="panel-body">
                    @foreach($users as $user)
                    <li class="list-grou-item">
                        <a href='/user/{{$user->id}}'>
                            {{$user->name}}
                        </a>
                    </li>
                    @endforeach
                </div>
            </div>
        </div>
    </div>
</div>
@endsection

// 25. how do we update the user info though?
// we go to the edit function to show all the user info first, then to update function
$user = User::find($id);
print_r($id);
return view('edit', compact('user'));

// 26. create edit.blad.php:
@extends('layouts.app')

@section('content')
<div class="container">
    <div class="row">
        <div class="col-md-10 col-md-offset-1">
            <div class="panel panel-default">
                <div class="panel-heading">User Info</div>

                <div class="panel-body">
                    <form action="/user/{{$user->id}}" method="POST">
                        Name: <input type="text" value="{{$user->name}}" />
                    </form>
                </div>
            </div>
        </div>
    </div>
</div>
@endsection

//27. To actually update the database, in usercontroller public function update():

public function update(Request $request, $id)
{
//
var_export($id);
var_export($request->name);
$user = User::find($id);
if($request->name != '')
    {
        $user->name = $request->name;
    }
    $user->save();

    return redirect()->action('UserController@show', [$id]);
}

// 28. Make another model to have snippets that belong to the user. FOR HOMEWORK: WILL MAKE A MEALS MODEL. in terminal:
php artisan make:model Snippets --migration

// 29. go to database>>migrations>>new migration will be in there(creat_snippets_table.php), add the following (around line 17 and 18):
public function up()
    {
        Schema::create('snippets', function (Blueprint $table) {
            $table->increments('id');
            $table->integer('user_id');
            $table->text('snippet');
            $table->timestamps();
        });
    }

// 30. run the migration:
php artisan migrate

**MAY NEED TO DROP ALL TABLES in order to migrate

// 31. seed again
php artisan db:seed

// 32. now we will make a relationship where a snippet belongs to a certain user
go to app>>Snippets.php
Line 10:

public function user(){
	return $this->belongsTo(User::class);
}

// 33. in this same file we will see "namespace App;" which lets us just use "User::class" in stead of "\App/User".

// 34. A user has many snippets potentially, so we have to do the reverse of what we just did. Go to the User.php model. Line 27 add:
public function snippets(){
    return $this->hasMany(Snippets::class);
}

^^no semi-colon at the end of that

// 35. Go into MySQL, into the snippets table, add a couple of snippets to user 1.

// 36. go to UserController. in public function show($id) add (line 56):

$user = User::find($id);
$snippets = $user->snippets;
// print_r($snippets);

return view('user', compact('user', 'snippets'));

// 37. then update user.blade.php to look like :

@extends('layouts.app')

@section('content')
<div class="container">
    <div class="row">
        <div class="col-md-10 col-md-offset-1">
            <div class="panel panel-default">
                <div class="panel-heading">User Info</div>

                <div class="panel-body">
                    <p> {{ $user->name }}</p>
                    <p> {{ $user->email }}</p>

                    <ul>
                        @foreach($snippets as $snippet)
                            <li>
                                <p>{{$snippet->snippet}}</p>
                            </li>
                        @endforeach
                    </ul>
                </div>
            </div>
        </div>
    </div>
</div>
@endsection

