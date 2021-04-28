##Making a forgot password system (A otp will be sent to user and user can change password)
![](https://image.freepik.com/free-vector/forgot-password-concept-illustration_114360-1123.jpg)

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
7. Setting up Mail\PasswordResetNotification.php 
```
class PasswordResetNotification extends Mailable
{
    use Queueable, SerializesModels;

    /**
     * Create a new message instance.
     *
     * @return void
     */
    public function __construct($token)
    {
        $this->token = $token;
    }

    /**
     * Build the message.
     *
     * @return $this
     */
    public function build()
    {
        return $this->view('emails.resetpassword')->with(['token' => $this->token]);
    }
}
```
8. Setting up resetpassword.blade.php
```
<center>
    <h1>OTP</h1>
    <b>{{$token}}</b>
</center>
```
9. Creating route for createpassword redirection 
```
//in web.php

Route::get('createpassword', function(){
	return view('pages.createpassword');
});
```
10. Setting up create password form
```
//in createpassword.blade.php

<form class="form-row mt-4 mb-5 align-items-center" action="/createpassword" method="POST">
	@csrf
	<div class="form-group col-sm-12">
	  <label>Email Address:</label>
	  <input type="email" class="form-control" placeholder="" name="email">
	</div>
	<div class="form-group col-sm-12">
	    <label>New Password</label>
	    <input type="password" class="form-control" placeholder="" name="password">
	</div>
	<div class="form-group col-sm-12">
	    <label>OTP</label>
	    <input type="text" class="form-control" placeholder="" name="token">
	</div>
	<div class="col-sm-6">
	  <button type="submit" class="btn btn-primary btn-block">submit</button>
	</div>
</form>
```
11. Setting up route for createpassword form 
```
//in web.php

Route::post('/createpassword', [ControllersForgotpasswordController::class, 'createpassword']);
```
12. Setting up createpassword function 
```
//in ForgotPasswordController.php

public function createpassword(Request $request)
    {
        $user_token  = DB::table('table_resetpassword')->where('email', $request->email)->first(); 
        if (!$user_token) {
            return back()->withError('Invalid Email');
        }
        if($user_token->token == $request->token){
            User::where('email', $request->email)->update(['password' => bcrypt($request->password)]);
            DB::delete('delete from table_resetpassword where email = ?',[$user_token->email]);
            return redirect('/login')->withSuccess("Password Has Been Reset");
        }
        
        else{
            return back()->withError("Invalid OTP");
        }
    }
```
