---
layout: post
permalink: /workshop2020/feedback
---

<style>
 .form-group {
   padding: 3px;
   width: 80%;
 }
 .group {
   margin: 20px;
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

Thanks again for joining the workshop! We would be glad if you could fill out the following survey to give us some insights into how useful this workshop was for you and what we could improve/change in the future to make it better. All forms are optional, so please ignore questions which you do not want to answer.

<form class="form" accept-charset="UTF-8" action="https://rse.pik-potsdam.de/magpie/registration/2020feedback.php" method="POST" enctype="multipart/form-data">
<div class="group">
  <div class="form-group">
    <label>Region</label>
    <select name="Region">
       <option>not saying</option>
       <option>Africa</option>
       <option>Asia</option>
       <option>Europe</option>
       <option>North America</option>
       <option>South America</option>
       <option>Oceania</option>
    </select>
  </div>
  <div class="form-group">
    <label>Position</label>
    <select name="Position">
       <option>not saying</option>
       <option>student</option>
       <option>PhD student</option>
       <option>PostDoc</option>
       <option>Professor</option>
       <option>Other</option>
    </select>
  </div>
  <div class="form-group">
    <label>Affiliation Type</label>
    <select name="AffiliationType">
       <option>not saying</option>
       <option>University</option>
       <option>Research Institute</option>
       <option>Company</option>
       <option>International Body</option>
       <option>Other</option>
    </select>
  </div>
</div>
<div class="group">
  <div class="form-group">
  Please specify how much you agree or disagree with the following statements:
  <br> (0 = disagree | 10 = agree)</div>
  <div class="form-group">
    <label>I were able to follow the steps in the tutorials</label>
    <br>0<input type="range" id="vol" name="Q1Follow" min="0" max="10" step="1">10
  </div>
  <div class="form-group">
  <div class="form-group">
    <label>I had all support I needed during the workshop</label>
    <br>0<input type="range" id="vol" name="Q2Support" min="0" max="10" step="1">10
  </div>
  <div class="form-group">
    <label>Some sessions were too fast</label>
    <br>0<input type="range" id="vol" name="Q3Fast" min="0" max="10" step="1">10
  </div>
  <div class="form-group">
    <label>Some sessions were too slow</label>
    <br>0<input type="range" id="vol" name="Q4Slow" min="0" max="10" step="1">10
  </div>
  <div class="form-group">
    <label>The workshop materials on GitHub were very helpful</label>
    <br>0<input type="range" id="vol" name="Q5Materials" min="0" max="10" step="1">10
  </div>
  <div class="form-group">
    <label>The workshop sessions were very helpful</label>
    <br>0<input type="range" id="vol" name="Q6Sessions" min="0" max="10" step="1">10
  </div>
  <div class="form-group">
    <label>I understand the MAgPIE framework now better</label>
    <br>0<input type="range" id="vol" name="Q7Understanding" min="0" max="10" step="1">10
  </div>
  <div class="form-group">
    <label>I am now able to run MAgPIE</label>
    <br>0<input type="range" id="vol" name="Q8Run" min="0" max="10" step="1">10
  </div>
  <div class="form-group">
    <label>I would recommend the workshop to colleagues</label>
    <br>0<input type="range" id="vol" name="Q9Recommend" min="0" max="10" step="1">10
  </div>
  <div class="form-group">
    <label>I have worked with MAgPIE in the past</label>
    <br>0<input type="range" id="vol" name="Q10Past" min="0" max="10" step="1">10
  </div>
  <div class="form-group">
    <label>I plan to work with MAgPIE in the future</label>
    <br>0<input type="range" id="vol" name="Q11Future" min="0" max="10" step="1">10
  </div>
</div>
</div>
<div class="group">
  <div class="form-group">
  Did you gain experience during the workshop?
  <br> (0 = low | 10 = high)</div>
  <div class="form-group">
    <label>My level of experience with MAgPIE before the workshop</label>
    <br>0<input type="range" id="vol" name="Q12ExperienceBefore" min="0" max="10" step="1">10
  </div>
  <div class="form-group">
  <div class="form-group">
    <label>My level of experience with MAgPIE after the workshop</label>
    <br>0<input type="range" id="vol" name="Q13ExperienceAfter" min="0" max="10" step="1">10
  </div>
</div>
</div>
<div class="group">
<div class="form-group">
  What did you like about the workshop?
<textarea id="like" name="like" rows="4" cols="50"> </textarea>
</div>
<div class="form-group">
  What should we make different next time?
<textarea id="suggestion" name="suggestion" rows="4" cols="50"> </textarea>
</div>
<div class="form-group">
  Anything else you want to tell us?
<textarea id="comment" name="comment" rows="4" cols="50"> </textarea>
</div>
</div>
  <button type="submit" class="btn btn-primary">Submit</button>
</form>
