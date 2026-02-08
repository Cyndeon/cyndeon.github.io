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
            Hi, I'm Justin Comans!<br><br>
            
            I'm a {{ age }}-year-old programmer specializing in plugin development and SDK integration for Unreal Engine 5. I've published tools to FAB Marketplace, led small programming teams, and built systems that make third-party services accessible through Blueprint nodes.<br><br>
            
            My focus is on practical tools: achievement systems that sync across Steam and Epic, editor extensions that improve workflows, and Blueprint APIs that designers can actually use. For projects meant to be reused or shared, I make sure to write proper documentation because nobody wants to reverse-engineer how something works.<br><br>
            
            When I'm not building plugins, I'm probably hunting achievements in games, which, coincidentally, inspired my achievement plugin.<br><br>
            
            I'm also passionate about VR development and hope to make it my primary focus professionally. Ever since I got my Rift S in 2019, I've been obsessed with VR. I've worked on some VR projects throughout my education. From my first school project with two classmates, to a VR-focused internship, to integrating OpenXR into a custom C++ engine.
        </p>
    </div>
</div>


<!-- Experience List -->
<h2>Technical Skills:</h2>
<ul style="font-size: 20px; line-height: 1.6;">
    <li><strong>Engines:</strong> Unreal Engine 5, Unity (2D/3D)</li>
    <li><strong>Languages:</strong> C++, C#, Blueprint</li>
    <li><strong>SDK Integration:</strong> Steamworks, Epic Online Services (EOS), OpenXR</li>
    <li><strong>Plugin Development:</strong> UE5 plugin architecture, Blueprint API design, editor tools</li>
    <li><strong>Code Porting:</strong> Converting C++ codebases to UE5 (types, threading, headers)</li>
    <li><strong>UE5 Systems:</strong> Subsystems (UEngineSubsystem), save systems, multi-threading (FRunnable)</li>
    <li><strong>Gameplay Systems:</strong> State machines, async operations, UI management</li>
</ul>

<h2>Professional Skills:</h2>
<ul style="font-size: 20px; line-height: 1.6;">
    <li>Team Leadership (Lead Programmer experience)</li>
    <li>Cross-discipline Collaboration</li>
    <li>Technical Documentation & Tutorial Writing</li>
    <li>Marketplace Publishing (FAB)</li>
    <li>Fluent in English & Dutch</li>
</ul>
</ul>

<!--Make icon redirect to the Portfolio page, don't touch this!-->
<script> document.querySelectorAll('#sidebar #avatar, #sidebar .site-title').forEach(function(link) { link.href = '/portfolio/'; }); </script>