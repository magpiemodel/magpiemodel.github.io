---
layout: page
permalink: /workshop2020/registration/
---

<style>
 .form {
   padding: 10px;
   width: 80%;
 }
 .privacy {
   padding: 10px;
   margin: 20px 20px 10px 20px;
   background-color: lightgrey;
   font-size: 90%;
 }
 .input {
   width: 100%;
 }
</style>
<form class="form" accept-charset="UTF-8" action="https://rse.pik-potsdam.de/magpie/registration/2020.php" method="POST" enctype="multipart/form-data" target="_blank">

  <div class="form-group">
    <label>Name<sup>*</sup></label>
    <select name="title">
       <option></option>
       <option>Ms</option>
       <option>Mr</option>
       <option>Dr</option>
       <option>Prof</option>
    </select>
    <input class="input" type="text" name="first_name" placeholder="First Name" required>
    <input class="input" type="text" name="last_name" placeholder="Last Name" required>
  </div>
  <div class="form-group">
    <label>Email address<sup>*</sup></label>
    <input class="input" type="email" name="email" placeholder="E-Mail" required>
  </div>
  <div class="form-group">
    <label>Affiliation<sup>*</sup></label>
    <input class="input" type="text" name="affiliation" placeholder="Affiliation" required>
  </div>
  <div class="form-group">
    <label>City</label>
    <input class="input" type="text" name="city" placeholder="City" required>
  </div>
  <div class="form-group">
    <label>Country</label>
    <input class="input" type="text" name="country" placeholder="Country" required>
  </div>

  <div class="privacy">
  <div> Privacy policy statement </div>
  By registering you give us consent to store your contact information and to use it to get in contact with you.
  We keep this information (participation details) until one month after the workshop. Upon request your data can be removed earlier.
  You have the right to access your data anytime by contacting us.
  </div>
  <div class="form-group">
    <input type="checkbox" name="dataprotectionpolicy" required>
    <label>I agree to the privacy policy</label>
  </div>

  <button type="submit" class="btn btn-primary">Submit</button>
</form>
