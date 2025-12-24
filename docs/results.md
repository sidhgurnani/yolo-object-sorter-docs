## General Remarks

Overall, I am satisfied with both aspects of the prototype as is (hardware and software), and it does ultimately serve the purpose that it was designed for. A great proof of concept has been reached and this is something that can be tweaked and applied across various areas. Examples include detecting bullet casings, various kinds of beans, resistors, LEGOs, etc.

If I were to iterate and make a new version, here is what I would do:

1. Software
    * Do more user testing to fix all bugs
    * Tweak Arduino code to ensure it counts total objects instead of total servo sweeps
    * Include relative file paths to make the user experience of setting up the sorting session a lot faster
    * Develop a system to better store data, as having the data stored in excel files is great for smaller scale projects and demos but horrible for large scale implementation
    * Create a few performance modes: conservative, normal, and rapid, all of which adjust the necessary settings to easily speed up and slow down the whole system. As of right now, the user would have to go into the code and find those parameters to change.
    * Create a mode that allows user easily do a dummy run to record a video of objects that need to be sorted (for the purposes of model training). This can be achieved by connecting a button to the Arduino, and if that gets pressed, then do the following: 
        * Start servo from neutral position
        * Sweep to Target in after 3 seconds
        * Sweep back to neutral after 3 seconds
        * Sweep to Not Target after 3 seconds
        * Sweep back to neutral after 3 seconds
        * Repeat until user hits button again
2. Hardware
    * Create channels for wires to easily go through and stay hidden
    * Design a place to hold all electronics that is out of the way from the bins, ideally underneath or somewhere off to the side in a separate 'black box'
    * Redesign some aspects to make it easier to assemble, specifically for tool access
    * Adjust mounting of the LCD to better fit the screen, and maybe hide away the green edge
    * Find a way to mount the potentiometer on the shell to allow for an easy way to adjust screen dimming
    * Design a flashlight mount or add a small strip of LED lights to better illuminate the platform

Here is what I might change if I had to do this all over again:

* **Source a different camera:** The camera used turned out to be pretty poor. The resolution wasn't the greatest, and that led to a harder time detecting complex objects such as coins. I would suggest either using a better camera or even allowing for smartphone compatibility.
* **Design a Web App:** This would allow for such a device to be run on much lower end devices at higher performances, but that would unlock new challenges with communication between the web interface and Arduino.
* **Switch away from a pure binary concept:** If the user knew the number of unique classes beforehand, then it would be entirely possible to have a modular system of bins that allows the number of bins to be specified. From there, only 1 pass would be required to sort through all objects, and would greatly speed up the sorting process. This has potential because during testing, the same working conditions between a human and the prototype always saw the prototype faster at sorting. However, a system with 10 bins would be hard to design. A middle ground is a 3 bin system, where 2 targets are being searched and the rest go to an other bin, which would halve the number of passes required.
* **Conciously slim down the size of the prototype:** While it is simple, the proptotype is rather large, and I think that the size of it can be trimmed down to help reduce filament usage and cost.
* **Better overall camera housing module:** The camera housing module was designed purely from an optimization point of view to use the least amount of filament. However, I would design an enclosed box with bright LED lights that illuminates the area where the objects are being viewed.
* **Platform geometry/size:** The platform itself is a restriction of what types of objects can be sorted. Bigger objects cannot be ran through simply because of the size of the platform.
* **Experiement with a different microcontroller:** Since the web app concept was shelved midway through the project, the need for a WebUSB board was not required. It might have not been a bad idea to experiement with a microcontroller that has an embedded camera to reduce some of the complexities in the design. This might also open up the path to try and use something like TinyML to optimize the sorting process and have it run directly on the microcontroller rather than using a laptop as the brain of the whole operation.
* **Design a system for object feeding:** To improve the user experience further, a system that can feed most kinds of objects should be introduced to make the process more hands-off.

If I were to redesign or continue forth with the project development, I would use the above list as a way to establish a good path forward to continue forth with the development. 

## My take on using GenAI for such projects

Overall, GenAI has been an incredible advancement in technology. The ability to put in what you want to do in words and have a computer translate that into an actual command is a huge leap. However, it does have its limitations, and to say this could be used to generate whole projects is not the best use of GenAI.

GenAI is like a tool, but not a simple tool like a pencil or pen. It is like a swiss army knife, where there are a ton of tools that can be used within this larger tool. In the same way that swiss army knives have knives, screwdrivers, or pliers, GenAI has the ability to interpret long documents, perform deep research, and generate code. No matter how impressive any swiss army knife is, a general contractor won't ask all their employees to start building homes only using swiss army knives. That is exactly the way I see GenAI. 

Even if I had prompted any GenAI model to build a project such as this one, it would not give me a working output at once. I might have to spend time debugging, or tweaking some functions, or maybe I gave it too much freedom in one aspect and too little freedom in another. At the end of the day, whole projects are not built in one step, or one prompt. They require many tools and in most cases, many people along the way. 

I will be transparent and say that I did use GenAI to work through this project, and utilized a combination of OpenAI's ChatGPT and Anthropic's Claude models to work through the project. However, my use of GenAI was purely to help supplement areas where I am not as strong. I did all of the idea generation and CAD on my own, but I used it to help me work through code. Note that I spent the time learning about how to code during my undergrad and again at the start of the project, which helped me immensely with the debugging process as well as sometimes guiding the GenAI model towards a more correct path. 