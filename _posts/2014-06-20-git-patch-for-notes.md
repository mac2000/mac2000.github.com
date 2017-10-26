---
layout: post
title: Git patch for notes
tags: [git, patch, apply, laravel, auth]
---

Lets assume that we are playing with something and want it to be saved for later use.

There is many options, from simples - copying files and saving them, to using git patches.

Here is what it looks like:

1. Setup your environment
2. Init git repository on it and commit all files
3. Switch to a new branch
4. Develop your sample code
5. Create path

From now, you have patch that will allow you to reproduce all changes on new installation.

Suppose you are configured your environment, run:

```sh
composer create-project laravel/laravel lara
cd lara
```

Configure database, application etc, and:

```sh
git add .
git commit -m 'init'
git checkout -b feature
```

Make your changes

```sh
git add .
git commit -m 'init'
```

Now you can export your changes to patch:

```sh
git format-patch master --stdout --shortstat --ignore-space-at-eol --ignore-space-change --ignore-all-space --ignore-blank-lines > ../auth.patch
```

On other hand to apply patch you can run:

```sh
git apply --ignore-space-change --ignore-whitespace ../auth.patch
```

Add `--check` option to see is it possible to apply patch at all.

Here salme of patch file:

```
{% raw %}
From 954554382f091fdbe10834edc6c28264fb516ff6 Mon Sep 17 00:00:00 2001
From: Marchenko Alexandr <marchenko.alexandr@gmail.com>
Date: Fri, 20 Jun 2014 15:41:26 +0300
Subject: [PATCH] init


    8 files changed, 222 insertions(+), 2 deletions(-)

diff --git a/app/database/migrations/2014_06_19_124526_create_users_table.php b/app/database/migrations/2014_06_19_124526_create_users_table.php
new file mode 100644
index 0000000..8f695f3
--- /dev/null
+++ b/app/database/migrations/2014_06_19_124526_create_users_table.php
@@ -0,0 +1,23 @@
+<?php
+
+use Illuminate\Database\Schema\Blueprint;
+use Illuminate\Database\Migrations\Migration;
+
+class CreateUsersTable extends Migration
+{
+    public function up()
+    {
+        Schema::create('users', function(Blueprint $table){
+            $table->increments('id');
+            $table->string('email')->unique();
+            $table->string('password');
+            $table->string('remember_token', 100)->nullable();
+            $table->timestamps();
+        });
+    }
+
+    public function down()
+    {
+        Schema::drop('users');
+    }
+}
diff --git a/app/database/seeds/DatabaseSeeder.php b/app/database/seeds/DatabaseSeeder.php
index 1989252..01cc3fd 100644
--- a/app/database/seeds/DatabaseSeeder.php
+++ b/app/database/seeds/DatabaseSeeder.php
@@ -11,7 +11,8 @@ class DatabaseSeeder extends Seeder {
        {
            Eloquent::unguard();

-       // $this->call('UserTableSeeder');
+        $this->call('UserTableSeeder');
+        $this->command->info('User table seeded!');
        }

    }
diff --git a/app/database/seeds/UserTableSeeder.php b/app/database/seeds/UserTableSeeder.php
new file mode 100644
index 0000000..43816a1
--- /dev/null
+++ b/app/database/seeds/UserTableSeeder.php
@@ -0,0 +1,11 @@
+<?php
+
+class UserTableSeeder extends Seeder
+{
+    public function run()
+    {
+        DB::table('users')->delete();
+
+        User::create(['email' => 'admin@example.com', 'password' => Hash::make('admin')]);
+    }
+}
diff --git a/app/routes.php b/app/routes.php
index 3e10dcf..c27e168 100644
--- a/app/routes.php
+++ b/app/routes.php
@@ -13,5 +13,66 @@

    Route::get('/', function()
    {
-   return View::make('hello');
+    return View::make('home');
+});
+
+Route::get('login', function()
+{
+    return View::make('login');
+});
+
+Route::post('login', function()
+{
+    $credentials = ['email' => Input::get('email'), 'password'=>Input::get('password')];
+
+    if (Auth::attempt($credentials, Input::get('remember', false))) {
+        return Redirect::to('/')->with('message', 'You are now logged in!');
+    } else {
+        return Redirect::to('login')->with('message', 'Your username/password combination was incorrect')->withInput();
+    }
+});
+
+Route::get('logout', function()
+{
+    Auth::logout();
+    return Redirect::to('/')->with('message', 'Your are now logged out!');
+});
+
+Route::post('logout', function()
+{
+    Auth::logout();
+    return Redirect::to('/')->with('message', 'Your are now logged out!');
+});
+
+Route::get('register', function()
+{
+    return View::make('register');
+});
+
+Route::post('register', function()
+{
+    $rules = [
+    'email' => 'required|email|unique:users',
+    'password' => 'required|alpha_num|between:1,255|confirmed',
+    'password_confirmation' => 'required|alpha_num|between:1,255'
+    ];
+
+    $validator = Validator::make(Input::all(), $rules);
+
+    if($validator->passes()) {
+        $user = new User;
+
+        $user->email = Input::get('email');
+        $user->password = Hash::make(Input::get('password'));
+
+        $user->save();
+
+        //return Redirect::to('login')->with('message', 'Thanks for registering!');
+
+        Auth::loginUsingId($user->id);
+        return Redirect::to('/')->with('message', 'Thanks for registering!');
+
+    } else {
+        return Redirect::to('register')->with('message', 'The following errors occurred')->withErrors($validator)->withInput();
+    }
    });
diff --git a/app/tests/AuthTest.php b/app/tests/AuthTest.php
new file mode 100644
index 0000000..1871600
--- /dev/null
+++ b/app/tests/AuthTest.php
@@ -0,0 +1,66 @@
+<?php
+
+class AuthTest extends TestCase
+{
+    public function setUp()
+    {
+        parent::setUp();
+        Artisan::call('migrate');
+        $this->seed();
+    }
+
+    public function testNotAuthorizedUserShouldSeeLoginButton()
+    {
+        $crawler = $this->client->request('GET', '/');
+
+        $this->assertResponseOk();
+        $this->assertCount(1, $crawler->filter('a:contains("Login")'));
+    }
+
+    public function testAuthorizedUserShouldSeeLogoutButton()
+    {
+        $user = new User;
+        $user->email = 'admin@example.com';
+        $this->be($user);
+
+        $crawler = $this->client->request('GET', '/');
+
+        $this->assertResponseOk();
+        $this->assertCount(1, $crawler->filter('a:contains("Logout")'));
+    }
+
+    public function testLoginFormValidation()
+    {
+        $this->call('POST', 'login', ['email' => '', 'password' => '']);
+
+        $this->assertRedirectedTo('login');
+        $this->assertSessionHas('message');
+    }
+
+    public function testRegisterFormValidation()
+    {
+        $this->call('POST', 'register', ['email' => '', 'password' => '', 'password_confirmation' => '']);
+
+        $this->assertRedirectedTo('register');
+        $this->assertSessionHas('message');
+        $this->assertSessionHasErrors();
+    }
+
+    public function testRegistration()
+    {
+        $this->call('POST', 'register', ['email' => 'temp@temp.com', 'password' => '123', 'password_confirmation' => '123']);
+
+        $this->assertRedirectedTo('/');
+        $this->assertSessionHas('message');
+    }
+
+    public function testLogin()
+    {
+        User::create(['email' => 'user@example.com', 'password' => Hash::make('user')]);
+
+        $this->call('POST', 'login', ['email' => 'user@example.com', 'password' => 'user']);
+
+        $this->assertRedirectedTo('/');
+        $this->assertSessionHas('message');
+    }
+}
diff --git a/app/views/home.blade.php b/app/views/home.blade.php
new file mode 100644
index 0000000..9157953
--- /dev/null
+++ b/app/views/home.blade.php
@@ -0,0 +1,17 @@
+@if(Session::has('message'))
+<p>{{ Session::get('message') }}</p>
+@endif
+
+<p>
+    @if(!Auth::check())
+    {{ HTML::link('register', 'Register') }}
+    {{ HTML::link('login', 'Login') }}
+    @else
+    {{ HTML::link('logout', 'Logout') }}
+
+    {{ Form::open(['url' => 'logout', 'method' => 'post']) }}
+    {{ Form::submit('Logout') }}
+    {{ Form::close() }}
+
+    @endif
+</p>
diff --git a/app/views/login.blade.php b/app/views/login.blade.php
new file mode 100644
index 0000000..5472a5e
--- /dev/null
+++ b/app/views/login.blade.php
@@ -0,0 +1,17 @@
+@if(Session::has('message'))
+<p>{{ Session::get('message') }}</p>
+@endif
+
+{{ Form::open(['url' => 'login', 'method' => 'post']) }}
+
+{{ Form::label('email', 'Email') }}
+{{ Form::email('email', Input::old('email')) }}
+
+{{ Form::label('password', 'Password') }}
+{{ Form::password('password') }}
+
+{{ Form::checkbox('remember', true, Input::old('remember', true)) }}
+
+{{ Form::submit('Submit') }}
+
+{{ Form::close() }}
diff --git a/app/views/register.blade.php b/app/views/register.blade.php
new file mode 100644
index 0000000..ff93c1f
--- /dev/null
+++ b/app/views/register.blade.php
@@ -0,0 +1,24 @@
+@if(Session::has('message'))
+<p>{{ Session::get('message') }}</p>
+@endif
+
+<ul>
+    @foreach($errors->all() as $error)
+    <li>{{ $error }}</li>
+    @endforeach
+</ul>
+
+{{ Form::open(['url' => 'register', 'method' => 'post']) }}
+
+{{ Form::label('email', 'Email') }}
+{{ Form::email('email', Input::old('email')) }}
+
+{{ Form::label('password', 'Password') }}
+{{ Form::password('password') }}
+
+{{ Form::label('password_confirmation', 'Password confirmation') }}
+{{ Form::password('password_confirmation') }}
+
+{{ Form::submit('Submit') }}
+
+{{ Form::close() }}
--
1.9.2.msysgit.0
{% endraw %}
```

What is cool about this, is that you can write some kind of tool that will generate html from this...
