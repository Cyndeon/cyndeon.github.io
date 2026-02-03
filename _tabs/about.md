---
icon: fas fa-info-circle
order: 5
---
{% assign birth_date = '2003-03-04' | date: '%s' %}
{% assign now = 'now' | date: '%s' %}
{% assign seconds_in_year = 31556952 %}
{% assign age = now | minus: birth_date | divided_by: seconds_in_year | floor %}

<style>
.about-container {
    display: flex;
    align-items: center;
    padding: 0px;
    gap: 20px;
}

.about-image-left { width: 600px; min-width: 400px; height: 600px; object-fit: cover; border-radius: 8px; }

.about-text-container {
    max-width: 600px;
    line-height: 1.6;
    position: relative;
    padding-top: 250px;
}

.about-image-small {
    width: 300px;
    height: auto;
    position: absolute;
    top: 0;
    left: 50%;
    transform: translateX(-50%);
    border-radius: 8px;
}

.about-text {
    font-size: 20px;
    font-weight: bold;
}

/* Mobile styles */
@media (max-width: 900px) {
    .about-container {
        flex-direction: column;
        align-items: center;
    }
    
    .about-image-left {
        width: 100%;
        max-width: 400px;
        height: 500px;
    }
    
    .about-text-container {
        padding-top: 0;
        text-align: center;
    }
    
    .about-image-small {
        position: relative;
        left: 0;
        transform: none;
        margin: 0 auto 20px auto;
        display: block;
    }
}
</style>

<div class="about-container">
    <img src="/assets/AboutMe/DieselAndME.jpeg" alt="DieselTall" class="about-image-left">
    
    <div class="about-text-container">
        <img src="/assets/Justin.jpg" alt="Small Image" class="about-image-small">
        <p class="about-text">
            Hi, I'm Justin Comans, and welcome to my website!<br><br>
            At {{ age }} years old, I have a deep passion for programming and game development.<br><br>
            With experience in C# and C++, I've worked with both Unity and Unreal Engine 5 to bring many fantastic ideas to life.<br><br>
            I'm especially excited about VR development and always looking for new ways to create immersive and entertaining experiences.
        </p>
    </div>
</div>


<!-- Experience List -->
<h2>Skillset:</h2>
<ul style="font-size: 20px; line-height: 1.6;">
    <li>C# Development</li>
    <li>C++ Development</li>
    <li>Unity (3D and 2D)</li>
    <li> Vr game development</li>
    <li>OpenXR (mainly with Unity)</li>
    <li>Unreal Engine 5 - Plugins,  Tools and Gameplay</li>
    <li>Software Architecture</li>
    <br>
	<li> Teamwork</li>
	<li> Communication</li>
	<li> Lead Programmer</li>
	<li> Fluent English</li>
	<li> Fluent Dutch</li>
</ul>