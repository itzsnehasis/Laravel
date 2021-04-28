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
