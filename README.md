# Discord-Login
Simple auth for Discord.

Checks to see if the user exists or not.
```php
composer require socialiteproviders/discord
```

```php
\SocialiteProviders\Manager\ServiceProvider::class,
```

```php
Socialite' => \SocialiteProviders\Manager\ServiceProvider::class,
```

```php
Route::get('/home', 'HomeController@index')->name('home');
Route::get('auth/discord', 'Auth\RegisterController@redirectToProvider')->name('auth');
Route::get('auth/discord/callback', 'Auth\RegisterController@handleProviderCallback')->name('callback');
```

```php
     public function handleProviderCallback()
     {
         $userr = Socialite::driver('discord')->user();
         $authUser = $this->findOrCreateUser($userr);
         Auth::login($authUser, true);

         return view('home');
     }

     private function findOrCreateUser($userr){
       $userAccount = User::where('discord_id', $userr->id)->first();
       if($userAccount){
         return $userAccount;
       }
       $newUser = User::create([
         'name' => $userr->name,
         'avatar' => $userr->avatar,
         'discord_discrim' => $userr->discriminator,
         'discord_locale' => $userr->locale,
         'discord_id' => $userr->id,
         'balance' => 0,
         'email' => $userr->email,
         'password' => null,
       ]);
       return $newUser;
     }
     ```
     ```php
     protected $fillable = [
        'name', 'balance', 'email', 'password', 'avatar', 'discord_discrim', 'discord_locale', 'discord_id',
    ];```
    
   ```php
   public function up()
    {
        Schema::create('users', function (Blueprint $table) {
            $table->increments('id');
            $table->integer('role_id')->default(2);
            $table->string('name');
            $table->string('avatar')->nullable();
            $table->string('discord_discrim')->nullable();
            $table->string('discord_locale')->nullable();
            $table->string('discord_id')->nullable();
            $table->integer('balance')->default(0);
            $table->string('email')->nullable();
            $table->string('password')->nullable();
            $table->rememberToken();
            $table->timestamps();
        });
    }```
    
    
