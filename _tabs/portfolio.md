---
title: Portfolio
icon: fas fa-solid fa-circle-user # Choose an icon from FontAwesome
order: 1 # Adjust based on where you want it in the nav
---
<style>
    :root {
        --image-text-gap: 0px; /* Set to 0px to remove any space between image and text */
    }

    .portfolio-item img {
        margin-bottom: var(--image-text-gap); /* Apply the gap between the image and text using the variable */
        width: 200px;
        height: 200px;
        object-fit: cover;
    }
    .portfolio-item p {
        margin: 0;
        padding: 0;
        line-height: 1; /* Adjust line height for better spacing */
        font-size: 1.2rem; /* Increase text size */
    }
    .portfolio-item p a {
		text-decoration: none !important; /* Remove underline */ 
		color: white !important; /* Set the default text color */ 
		font-weight: bold !important; /* Ensure bold text */
    }
}
</style>

<p style="text-align: center; font-size: 14px; font-style: italic; margin-top: 10px;">
    Click on the text to go to the item's page
</p>

<!-- Container holding all the portfolio items -->
<div style="display: flex; flex-wrap: wrap; gap: 20px;">

    <!-- Portfolio Item -->
    <div class="portfolio-item" style="flex: 1 0 48%; text-align: center;">
	        <!-- Link to image source (+ text if it won't load)-->
            <img src="/assets/Thumbnails/Ortus.png" alt="OpenXR C++"  style="object-fit: contain !important">
        <!-- Link To site + Text -->
        <p><a href="https://buas.itch.io/team-cumin">
        Ortus (blog upcoming)
        </a></p>
    </div>
    
    <!-- Portfolio Item -->
    <div class="portfolio-item" style="flex: 1 0 48%; text-align: center;">
	        <!-- Link to image source (+ text if it won't load)-->
            <img src="/assets/Thumbnails/PatientZero.png" alt="OpenXR C++" >
        <!-- Link To site + Text -->
        <p><a href="https://itsnoonytime.itch.io/patient-0">
        Patient Zero (blog upcoming)
        </a></p>
    </div>
    
    <!-- Portfolio Item -->
    <div class="portfolio-item" style="flex: 1 0 48%; text-align: center;">
	        <!-- Link to image source (+ text if it won't load)-->
            <img src="/assets/Thumbnails/OpenXRCpp.png" alt="OpenXR C++" >
        <!-- Link To site + Text -->
        <p><a href="/posts/OpenXrCustomEngine/">
        OpenXR in C++ engine
        </a></p>
    </div>

    <!-- Portfolio Item -->
    <div class="portfolio-item" style="flex: 1 0 48%; text-align: center;">
	        <!-- Link to image source (+ text if it won't load)-->
            <img src="/assets/Thumbnails/UnityVrDemo.png" alt="OpenXR C++" >
        <!-- Link To site + Text -->
        <p><a href="https://gamejolt.com/games/VR-Minigame-Collection/602509">
        Unity VR Minigames (blog upcoming)
        </a></p>
    </div>

    <!-- Portfolio Item -->
    <div class="portfolio-item" style="flex: 1 0 48%; text-align: center;">
	        <!-- Link to image source (+ text if it won't load)-->
            <img src="/assets/Thumbnails/Beautiful.png" alt="OpenXR C++" >
        <!-- Link To site + Text -->
        <p><a href="https://gamejolt.com/games/Beautiful-text-adventure/473929">
        Beautiful - A text adventure (blog upcoming)
        </a></p>
    </div>
</div>
