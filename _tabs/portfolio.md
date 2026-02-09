---
icon: fas fa-solid fa-circle-user # Choose an icon from FontAwesome
order: 1 # Adjust based on where you want it in the nav
---
<link rel="stylesheet" href="/assets/css/cards.css">

<!-- Removes title at the top -->
<style>
h1.dynamic-title {
  display: none !important;
}
</style>

<!-- Removes right side bar -->
<style>
#panel-wrapper {
  display: none !important;
}
main.col-12.col-lg-11.col-xl-9 {
  max-width: 100% !important;
  flex: 0 0 100% !important;
}
</style>

<!-- Filter Button -->
<div class="filter-container" style="display: flex; flex-wrap: wrap; align-items: center; gap: 8px; margin-bottom: 0px; margin-top: -42px; margin-bottom: -12px">
  <button id="toggle-filters" style="background-color: #444; color: white; border: none; padding: 4px 8px; border-radius: 50px; cursor: pointer;">Filters ▼</button>

<!-- Specific Filter Buttons -->
<button class="filter-btn active" data-filter="all" style="display: none;">All</button>
  <!-- Visible Tags -->
  <button class="filter-btn" data-filter="blogpost">Blog</button>
  <button class="filter-btn" data-filter="unity">Unity</button>
  <button class="filter-btn" data-filter="unreal">Unreal</button>
  <button class="filter-btn" data-filter="cpp">C++</button>
  <button class="filter-btn" data-filter="csharp">C#</button>
  <button class="filter-btn" data-filter="virtual-reality">VR</button>
  <button class="filter-btn" data-filter="tools">Tools</button>
  <button class="filter-btn" data-filter="solo">Solo</button>
  <button class="filter-btn" data-filter="team">Team</button>
  <!-- Hidden Tags -->
  <button class="filter-btn" data-filter="gamejam">Gamejam</button>
  <button class="filter-btn" data-filter="lead">Lead</button>
</div>

<div class="projects-container">
<!-- Tag order, IMPORTANT TO FOLLOW! -->
<!-- Blog, Engine, Language, Other tags (in order of you want the user to see), Solo/Team -->

<!-- UE5 Achievement Plugin -->
<div class="card-wrapper highlight" style="--highlight-gradient-start: #960bdb; --highlight-gradient-end: #1535e8;" data-tags="tools lead">
  <div class="project-card highlight">
    <div class="card-inner">
      <div class="card-image">
        <img src="/assets/Thumbnails/UE5AchievementPlugin.png" alt="">
      </div>
      <div class="card-content">
        <h3>UE5 Achievement Plugin</h3>
        <p class="card-description">A custom achievement system plugin for Unreal Engine 5 with EOS and Steamworks support</p>
        <ul class="card-skills">
		<li>Plugin architecture</li>
		<li>SDK integration (Steamworks/EOS)</li>
		<li>Saving/loading system (using UE5's),</li>
		<li>UI Widgets</li>
		<li>Blueprint API design</li>
		<li>Cross-version compatibility (UE 5.5-5.7)</li>
        </ul>
        <div class="project-tags">
          <span class="project-tag blogpost">Blog</span>
          <span class="project-tag unreal">Unreal</span>
          <span class="project-tag cpp">C++</span>
          <span class="project-tag tools">Tools</span>
          <span class="project-tag solo">Solo</span>
        </div>
      </div>
    </div>
    <a href="/posts/UE5AchievementPlugin/" class="card-link"></a>
  </div>
</div>

<!-- Wanderland -->
<div class="card-wrapper highlight" style="--highlight-gradient-start: #f264f5; --highlight-gradient-end: #64a8f5;" data-tags="tools lead">
  <div class="project-card highlight">
    <div class="card-inner">
      <div class="card-image">
        <img src="/assets/Thumbnails/Wanderland.png" alt="">
      </div>
      <div class="card-content">
        <h3>Wanderland</h3>
        <p class="card-description">A magical experience where you fight a boss with physical wands in real life</p>
        <ul class="card-skills">
		<li>C++ to UE5 code conversion & plugin architecture </li>
		<li>Plugin architecture </li>
		<li>Multi-threading (FRunnable) </li>
		<li>Blueprint API design & exposure </li>
		<li>Subsystem architecture (UEngineSubsystem) </li>
		<li>State machine implementation </li>
        </ul>
        <div class="project-tags">
          <span class="project-tag blogpost">Blog</span>
          <span class="project-tag unreal">Unreal</span>
          <span class="project-tag cpp">C++</span>
          <span class="project-tag team">Team</span>
        </div>
      </div>
    </div>
    <a href="/posts/Wanderland/" class="card-link"></a>
  </div>
</div>

<!-- ORTUS -->
<div class="card-wrapper">
    <div class="project-card">
      <div class="card-image">
        <img src="/assets/Thumbnails/Ortus.png" alt="">
      </div>
      <div class="card-content">
        <h3>ORTUS</h3>
        <p class="card-description">A short but hectic and extremely fun top-down shooter</p>
        <div class="project-tags">
          <span class="project-tag unreal">Unreal</span>
          <span class="project-tag cpp">C++</span>
          <span class="project-tag team">Team</span>
        </div>
      </div>
      <a href="https://buas.itch.io/team-cumin" class="card-link" target="_blank"></a>
    </div>
</div>

<!-- Patient Zero -->
<div class="card-wrapper" data-tags="gamejam">
    <div class="project-card">
      <div class="card-image">
        <img src="/assets/Thumbnails/PatientZero.png" alt="">
      </div>
      <div class="card-content">
        <h3>Patient Zero</h3>
        <p class="card-description">As the first zombie ever, try to escape the facility and turn guards to your cause</p>
        <div class="project-tags">
          <span class="project-tag unity">Unity</span>
          <span class="project-tag csharp">C#</span>
          <span class="project-tag team">Team</span>
        </div>
      </div>
      <a href="https://itsnoonytime.itch.io/patient-0" class="card-link" target="_blank"></a>
    </div>
</div>

<!-- OpenXR -->
<div class="card-wrapper">
    <div class="project-card">
      <div class="card-image">
        <img src="/assets/Thumbnails/OpenXRCpp.png" alt="">
      </div>
      <div class="card-content">
        <h3>OpenXR</h3>
        <p class="card-description">A blog post explaining how I implemented OpenXR into a custom C++ engine made by BUAS</p>
        <div class="project-tags">
          <span class="project-tag blogpost">Blog</span>
          <span class="project-tag cpp">C++</span>
          <span class="project-tag virtual-reality">VR</span>
          <span class="project-tag solo">Solo</span>
        </div>
      </div>
      <a href="/posts/OpenXrCustomEngine/" class="card-link"></a>
    </div>
</div>

<!-- BrainInc -->
<div class="card-wrapper" data-tags="lead">
    <div class="project-card">
      <div class="card-image">
        <img src="/assets/Thumbnails/BrainInc.png" alt="">
      </div>
      <div class="card-content">
        <h3>BrainInc</h3>
        <p class="card-description">A game I was Lead on during my Internship in 2022 <br> Note: The page for the game no longer exists, likely cancelled. <br>
         [BLOG UPCOMING]</p>
        <div class="project-tags">
        <span class="project-tag unity">Unity</span>
          <span class="project-tag csharp">C#</span>
          <span class="project-tag virtual-reality">VR</span>
          <span class="project-tag team">Team</span>
        </div>
      </div>
      <a href="/posts/BrainInc" class="card-link"></a>
    </div>
</div>


<!-- VR Minigame Collection -->
<div class="card-wrapper">
    <div class="project-card">
      <div class="card-image">
        <img src="/assets/Thumbnails/UnityVrDemo.png" alt="">
      </div>
      <div class="card-content">
        <h3>VR Minigame Collection</h3>
        <p class="card-description">A small collection of VR minigames</p>
        <div class="project-tags">
          <span class="project-tag unity">Unity</span>
          <span class="project-tag csharp">C#</span>
          <span class="project-tag virtual-reality">VR</span>
          <span class="project-tag team">Team</span>
        </div>
      </div>
      <a href="https://gamejolt.com/games/VR-Minigame-Collection/602509" class="card-link" target="_blank"></a>
    </div>
</div>

<!-- Beautiful -->
<div class="card-wrapper">
    <div class="project-card">
      <div class="card-image">
        <img src="/assets/Thumbnails/Beautiful.png" alt="">
      </div>
      <div class="card-content">
        <h3>Beautiful - A text adventure</h3>
        <p class="card-description">An interactive text-based adventure game with numerous endings</p>
        <div class="project-tags">
          <span class="project-tag csharp">C#</span>
          <span class="project-tag solo">Solo</span>
        </div>
      </div>
      <a href="https://gamejolt.com/games/Beautiful-text-adventure/473929" class="card-link" target="_blank"></a>
    </div>
</div>

</div>

<!--Make icon redirect to About Me, don't touch this!-->
<script>
document.querySelectorAll("#sidebar #avatar, #sidebar .site-title").forEach(function(link) {
  link.href = "/about/";
});
</script>

<!-- Button Script -->
<script>
document.getElementById("toggle-filters").addEventListener("click", function() {
  var filters = document.querySelectorAll(".filter-btn");
  var isHidden = filters[0].style.display === "none";
  
  filters.forEach(function(btn) {
    btn.style.display = isHidden ? "inline-block" : "none";
  });
  
  this.textContent = isHidden ? "Filters ▲" : "Filters ▼";
  
  if (!isHidden) {
    document.querySelectorAll(".card-wrapper").forEach(function(card) {
      card.classList.remove("hidden");
    });
    
    document.querySelectorAll(".filter-btn").forEach(function(btn) {
      btn.classList.remove("active");
      if (btn.getAttribute("data-filter") === "all") {
        btn.classList.add("active");
      }
    });
  }
});

document.querySelectorAll(".filter-btn").forEach(function(btn) {
  btn.style.display = "none";
});
</script>

<!-- Filters script -->
<script>
document.querySelectorAll(".filter-btn").forEach(function(btn) {
  btn.addEventListener("click", function() {
    document.querySelectorAll(".filter-btn").forEach(function(b) {
      b.classList.remove("active");
    });
    this.classList.add("active");
    
    var filter = this.getAttribute("data-filter");
    
    document.querySelectorAll(".card-wrapper").forEach(function(card) {
      if (filter === "all") {
        card.classList.remove("hidden");
      } else {
        var dataTags = card.getAttribute("data-tags") || "";
        var hasDataTag = dataTags.split(" ").indexOf(filter) !== -1;
        var hasVisibleTag = card.querySelector(".project-tag." + filter);
        
        if (hasDataTag || hasVisibleTag) {
          card.classList.remove("hidden");
        } else {
          card.classList.add("hidden");
        }
      }
    });
  });
});
</script>