This page provides a little introduction to what is required by the prototype and how I aim to solve those requirements. 

## Requirements

First off, here is a list of basic requirements:

* Prototype must allow user to be able to load objects with relative ease, without having to worry too much about the orientation and the type of object loaded.
* Prototype must be able to work with, or show potential to work with, a variety of objects with differing geometry, weight, size, etc.
* Prototype must allow objects to be visible from the webcam and allow the YOLO software to be able to detect and sort objects.
* Prototype must accomodate a "target" and "other" bin, to allow for binary sorting.
* Prototype must allow all electronics to be easily accessible to user to allow for debugging and/or allow for any required work to be performed (rewiring, reconnecting, etc) while also not interfering with the sorting process.
* Prototype must contain a manual override to allow for human intervention in case of failure or unexpected errors.
* Prototype must be able to sort, or show potential to sort, with relative high accuracy.

<div><br></div>

**The list above details all basic requirements, however it leaves out a few desireable features, which would be implemented if the project timeline were to be extended. Ideally, the prototype will also contain:**

* **Error Handling &rarr;** System should have a way of knowing when something has gone wrong and attempt a correction.
* **Scalability &rarr;** System should be "plug and play", where there would not be much work to scale this up to whatever application it is required for.

<div><br></div>

## Final Prototype Solution (Ver 2.X -- 9/30/25)

After speaking with my project mentor, we came to the conclusion that the solution decided on in Version 1.X (Vibration Bowl Feeder + Conveyor Belt) is not ideal for the scope of the project. Pursuing that route would add a lot of complexities with regards to the object feeding, which is not super in line with the scope of the project. While an automated system to feed objects pretty much effortlessly is a solid idea, it is indeed out of scope. 

The main focus of the project is on object detection and using various models to be able to detect and sort objects. That being said, object feeding will be done manually and is not a part of the project I will be designing too much.

Here is the path forward with regards to the prototype: 

1. Use an external webcam to allow for greater flexibility. 
2. Create a standalone device that is not attached to the computer in any way (minus any external webcam connections via USB), so that any changes in laptop size and type can be negated and produce something more universal.

I am providing a brief concept sketch below to detail what the standalone device might look like: 

![Concept Sketch for Version 2.X](./assets/YOLO%20Sorter%20Concept%202.0.jpg)

Below I am doing a requirements check on the object feeding and object transportation, comparing the idea of a standalone device against the selected solutions from Version 1.X. 

### Object Feeding

What is being compared?

* Vibration Bowl Feeder
* Manual feeding into standalone device

| Requirement | Vibration Bowl Feeder | Manual Feeding | Notes / Explanation|
|---------------------------------------------------|-----------------------|----------------------|-------------------------------------------------------------|
| Easy loading of objects without worrying about orientation or type | ✔️ Meets | ✔️ Meets | Vibration Bowl Feeders can feedmore consistently and reliably (IF designed correctly); Manual Feeding is simpler and orientation can be controlled by user |
| Works with a variety of object sizes, weights, and shapes | ⚠️ Partially meets | ✔️ Meets | Viration Bowl Feeder is limited by the bowl geometry, but manual feeding can work with objects of any geometry (as long as the chute/ramp to feed is large enough) |
| Allows objects to be visible for YOLO detection | ➖ No effect | ➖ No effect | Object visibility depends on camera setup, independent of method of feeding |
| Accommodates “target” and “other” sorting bins | ➖ No effect | ➖ No effect | Sorting bins are outside the feeder design |
| Electronics easily accessible for debugging | ➖ No effect | ➖ No effect | Electronics housing will be designed separately |
| Manual override for human intervention | ➖ No effect | ➖ No effect | Manual override controlled elsewhere, not part of feeder |
| High accuracy sorting capability | ➖ No effect | ➖ No effect | Sorting accuracy depends on YOLO models |

Even from the table above, the manual feeding is simpler due to less complexity with mechanical components, and more importantly, vibrations. Designing for vibrations is inherently difficult, and even if the feeder was designed and could feed objects up, there will always be a type of object ill-suited to be fed down to either geometry or object's natural frequency. 

### Transporting Objects for Detection and Sorting

While a comparison can be drawn for the transportation of objects, something of note is that there are two totally different ideas in play. 

Version 1.X had the idea of taking objects and having be fed out and moved into view, however, as specified in the concept sketch for Version 2.X, objects will be fed by means of a gravity hopper or manual feeding directly into the view of the object. This induces much less mechanical complexity as a whole and thus there is a very clear, straightforward, and obvious path forward. 

### Comparison of Overall Concepts

Below is a brief description of both concepts. Note that it includes the object's point of view as it travels to the bins.

* Version 1.X: Vibration Bowl Feeder &rarr; 3D Printed Conveyor Belt &rarr; Webcam's FOV (viewable by using mirror to reorient FOV) &rarr; Bins
* Version 2.X: Manual Feeding (via hopper/ramp/chute) &rarr; Platform (within Webcam's FOV) &rarr; Bins

I will do two comparisons, one against the requirements as a whole and one that details prototype feasibilty relative to one another.

| Requirement | Version 1.X | Version 2.X | Notes / Explanation|
|---------------------------------------------------|-----------------------|----------------------|-------------------------------------------------------------|
| Easy loading of objects without worrying about orientation or type | ✔️ Meets, but with correct, finely tuned design | ✔️ Meets, but more user effort |  |
| Works with a variety of object sizes, weights, and shapes | ⚠️ Partially meets | ✔️ Meets | Version 1.X is limited by bowl geometry, while Version 2.X can work with a greater variety of objects |
| Allows objects to be visible for YOLO detection | ✔️ Meets | ✔️ Meets | Both concepts leverage a camera to be able to view and detect objects |
| Accommodates “target” and “other” sorting bins | ✔️ Meets | ✔️ Meets | Both accomodate bins to be able to sort objects |
| Electronics easily accessible for debugging | ✔️ Meets | ✔️ Meets | Electronics will be housed separately in both cases, but can be made easily accessible  |
| Manual override for human intervention | ✔️ Meets | ✔️ Meets | Manual override can be built in |
| High accuracy sorting capability | ✔️ Meets | ✔️ Meets | Sorting accuracy depends on YOLO models, which is able to be leveraged in both cases |

| Prototype Feasibility Requirement | Version 1.X | Version 2.X | Notes / Explanation|
|---------------------------------------------------|-----------------------|----------------------|-------------------------------------------------------------|
| Relative Mechanical Complexity | 🔴 More complex| 🟢 Less complex | |
| Relative Electrical Complexity | 🔴 More complex | 🟢 Less complex | Version 1.X will need a vibration motor and a DC motor to move the belt, Version 2.X will need a servo motor to control the platform |
| Relative Costs | 🔴 More | 🟢 Less | Both will cost a similar amount to put together, which includes costs of making and sourcing components|
| Prototype-able? | 🔴 Less prototype-able | 🟢 More prototype-able | Both concepts can be 3D printed and integrated, but Version 1.X requires more components that have to be carefully designed with vibrations in mind|
| Reproducible? | 🔴 Not really | 🟢 Yes, easily | Reasoning due to ability to prototype |
| Scalable | 🔴 Less scalable | 🟢 More scalable | Version 2.X is simpler, and will be easier to assemble and modify for larger sized objects |

## Considered Solutions (Ver 1.X -- 8/26/25)

**NOTE:** This is an older version of the prototype that is not being considered after 8/26/25. I have retained the information below for documentation and reference purposes.

Based on the list of basic requirements above, here are solutions being considered for the broader aspects of the prototype.

### Object Feeding

What is being considered?

* Vibration Bowl Feeder
* Centrifugal Feeder

| Requirement | Vibration Bowl Feeder | Centrifugal Feeder | Notes / Explanation|
|---------------------------------------------------|-----------------------|----------------------|-------------------------------------------------------------|
| Easy loading of objects without worrying about orientation or type | ✔️ Meets | ✔️ Meets | Vibration Bowl Feeders feed slower, but are more reliable; Centrifugal will require a stronger motor or a high torque gear train, is simpler, but is less universal |
| Works with a variety of object sizes, weights, and shapes | ⚠️ Partially meets | ⚠️ Partially meets | Both concepts are limited by bowl geometry to an extent, but not heavily affected by it for prototype purposes |
| Allows objects to be visible for YOLO detection | ➖ No effect | ➖ No effect | Object visibility depends on camera setup, independent of feeder |
| Accommodates “target” and “other” sorting bins | ➖ No effect | ➖ No effect | Sorting bins are outside the feeder design |
| Electronics easily accessible for debugging | ➖ No effect | ➖ No effect | Electronics housing will be designed separately |
| Manual override for human intervention | ➖ No effect | ➖ No effect | Manual override controlled elsewhere, not part of feeder |
| High accuracy sorting capability | ➖ No effect | ➖ No effect | Sorting accuracy depends on YOLO models |

When comparing both solutions on paper, there is no obvious standout choice to start with, I will also do a quick comparison below to highlight a path forward.

| Prototype Feasibility Requirement | Vibration Bowl Feeder | Centrifugal Feeder | Notes / Explanation|
|---------------------------------------------------|-----------------------|----------------------|-------------------------------------------------------------|
| Relative Mechanical Complexity | 🟢 Less complex | 🔴 More complex | Vibration Bowl Feeder will have less mechanical components to work with; Centrigufal Feeder will have more mechanical components to work with and has to be designed more carefully|
| Relative Electrical Complexity | 🔴 Slightly more complex | 🟢 Slightly less complex | Vibration Bowl Feeders will need extra tuning and sourcing of electrical components; Centrifugal Bowl Feeders will require less elctrical components and thus less complexity |
| Relative Costs | ⚪ Similar | ⚪ Similar | Both will cost a similar amount to put together, which includes costs of making and sourcing components|
| Prototype-able? | ⚪ Similar | ⚪ Similar | Both concepts can be 3D printed |
| Reproducible? | 🟢 Slightly More | 🔴 Slightly Less | Vibration Bowl Feeders have less parts to put together, which reduces chances of parts not fitting together after 3D printing |
| Scalable | ⚪ Similar | ⚪ Similar | Both concepts can be adapted and scaled in a post-project timeline to fit the needs required through changing out any components that were sourced and altering dimensions in CAD |

Both are pretty equal in terms of viablility, but the vibration bowl feeder is slightly less complex overall and has the potential to be more reproducible to other people. Therefore, I will stick to making a vibration bowl feeder for now.

### Transporting Objects for Detection and Sorting

This section deals with solutions being considered to handle moving the objects being sorted from the feeding mechanism to the webcam'e field of view and then to the bins. 

What is being considered?

* **Option 1:** System of Linear Tracks &rarr; Webcam's FOV (via a enclosure) &rarr; Bins
* **Option 2:** 3D Printed Conveyor Belt &rarr; Webcam's FOV (viewable by using mirror to reorient FOV) &rarr; Bins

| Requirement | Option 1 | Option 2 | Notes / Explanation|
|---------------------------------------------------|-----------------------|----------------------|-------------------------------------------------------------|
| Easy loading of objects without worrying about orientation or type | ⚠️ Potential negative | ✔️ Meets |  I will focus on loading in the context of going to webcam FOV and then to bins; Option 1 has to be designed carefully to accomodate a vast amount of objects; Option 2 is more universal and doesn't deoend on orientation or type |
| Works with a variety of object sizes, weights, and shapes | ⚠️ Potential Negative | ✔️ Meets | Option 1 is limited in terms of track geometry; Option 2 more universal |
| Allows objects to be visible for YOLO detection | ⚠️ Potential Negative | ✔️ Meets | Option 1's enclosure has to designed to either allow in natural ambient light or be well-lit; Option 2 uses ambient lighting, which works reasonably well |
| Accommodates “target” and “other” sorting bins | ✔️ Meets | ✔️ Meets | Sorting bins can be integrated into both solutions |
| Electronics easily accessible for debugging | ➖ No effect | ➖ No effect | Electronics housing will be designed separately |
| Manual override for human intervention | ➖ No effect | ➖ No effect | Manual override controlled elsewhere, not part transportation sub-system |
| High accuracy sorting capability | ➖ No effect | ➖ No effect | Sorting accuracy depends on YOLO models |

Based on requirements alone, there is a clear path forward. At worst, a conveyor belt will be able to provide more proof of concept than the linear track system, and at best, it will be a sub-system that is more universal. No feasibility study will be conducted unless something goes wrong with the conveyor belt concept.