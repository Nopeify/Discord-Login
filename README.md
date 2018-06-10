# Discord-Login
Simple Larvel auth for Discord. You're welcome.

+ Checks to see if the user exists or not.
+ IF exists, login the user.
+ If NOT - make user account using Discord api's callback json stuff.
  
Console
```php
composer require socialiteproviders/discord
php artisan make:auth
```

App.php  
```php
\SocialiteProviders\Manager\ServiceProvider::class,
  
...
  
Socialite' => \SocialiteProviders\Manager\ServiceProvider::class,
```
Web.php  
```php
Route::get('/home', 'HomeController@index')->name('home');
Route::get('auth/discord', 'Auth\RegisterController@redirectToProvider')->name('auth');
Route::get('auth/discord/callback', 'Auth\RegisterController@handleProviderCallback')->name('callback');
```
EventServiceProvider.php  
```php
protected $listen = [
        'App\Events\Event' => [
            'App\Listeners\EventListener',
        ],
        \SocialiteProviders\Manager\SocialiteWasCalled::class => [
          'SocialiteProviders\\Discord\\DiscordExtendSocialite@handle',
        ],
    ];
```
services.php  
```php
'discord' => [
      'client_id' => env('DISCORD_KEY'),
      'client_secret' => env('DISCORD_SECRET'),
      'redirect' => env('DISCORD_REDIRECT_URI'),
  ],
```
.env  
```php
DISCORD_KEY=
DISCORD_SECRET=
DISCORD_REDIRECT_URI=
```
RegisterController.php  
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
User.php  
```php
     protected $fillable = [
        'name', 'balance', 'email', 'password', 'avatar', 'discord_discrim', 'discord_locale', 'discord_id',
    ];
```

2014_10_12_000000_create_users_table.php
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
    }
```
    
\vendor\socialiteproviders\discord\provider.php
```php
protected $scopes = [
        'identify',
        'email',
        'connections', // custom
        'guilds', // custom

    ];
...
protected function mapUserToObject(array $user)
    {
        return (new User())->setRaw($user)->map([
            'id' => $user['id'],
            'nickname' => sprintf('%s#%s', $user['username'], $user['discriminator']),
            'name' => $user['username'],
            'email' => $user['email'],
            'avatar' => (is_null($user['avatar'])) ? null : sprintf('https://cdn.discordapp.com/avatars/%s/%s.jpg', $user['id'], $user['avatar']),
            'discriminator' => $user['discriminator'],
            'locale' => $user['locale'],
        ]);
    }
```

```
http://localhost/auth/discord -> Redirect
http://localhost/auth/discord/callback? -> Callback
```
