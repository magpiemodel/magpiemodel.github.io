---
layout: post
permalink: /magpie22/register
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

 .form-group {
   padding: 5px;
 }
</style>

Welcome to the MAgPIE22 workshop & stories registration page. Please enter your details and specify which sessions you intent to join. As workshop slots will be more limited than stories slots we would like to encourage you only to select the workshop sessions if you really intent to join them.

Please be prepared that it may take a few days before you get your registration confirmed via mail. Please contact us via <magpie@pik-potsdam.de> in case you do not receive a confirmation mail within 2 weeks after registration.

<form class="form" accept-charset="UTF-8" action="https://rse.pik-potsdam.de/magpie/registration/2022.php" method="POST" enctype="multipart/form-data">

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

  <div class="form-group">
    <div class="form-group">
      <label>Which sessions to you plan to attend?</label>
    </div>
    <input type="checkbox" name="participate_workshop">
    <label>MAgPIE introduction workshop</label>
    <input type="checkbox" name="participate_stories">
    <label>MAgPIE stories</label>
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
