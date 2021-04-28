## Making a forgot password system (A otp will be sent to user and user can change password)

1. Setting up forgot password link and route
```
//in login.blade.php

<a href="/forgotpassword">Forgot Password</a>
```
```
//in web.php

Route::get('/forgotpassword', function(){
	return view('pages.forgotpassword');
});
```
2. Setting up forgot password form
```
//in forgotpassword.blade.php

<form class="form-row mt-4 mb-5 align-items-center" action="/forgotpassword" method="POST">
	@csrf
	<div class="form-group col-sm-12">
	  <label>Email Address:</label>
	  <input type="email" class="form-control" placeholder="" name="email">
	</div>
	<div class="col-sm-6">
	  <button type="submit" class="btn btn-primary btn-block">submit</button>
	</div>
</form>
```
3. Setting up route for forgotpassword form submission 
```
//in web.php

Route::post('/forgotpassword', [ControllersForgotpasswordController::class, 'index']);
```
4. Setting up a new Table 
```
//in create_resetpassword_migration.php

public function up()
    {
        Schema::create('table_resetpassword', function (Blueprint $table) {
            $table->string('email')->primary();
            $table->string('token');
            $table->timestamps();
        });
    }
```
6. Making index function 
```
//in ForgotPasswordController.php

public function index(Request $request)
    {

        //check the user exist or not in database
        $user = User::where('email', $request->email)->first();
        if (!$user) {
            return back()->withError('Invalid Email');
        }

        //inserting data with the token 
        DB::table('table_resetpassword')->insert([
            'email' => $request->email,
            'token' => str_random(5)
        ]);

        //get the token just created above
        $token_get = DB::table('table_resetpassword')->where('email', $request->email)->first();

        //send link to user
        Mail::to($user->email)->send(new PasswordResetNotification($token_get->token));
        return redirect('createpassword')->withSuccess("OTP Has Been Sent To Your Email");
    }
```
7. Setting up Mail\
