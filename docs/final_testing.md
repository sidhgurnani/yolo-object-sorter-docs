## Testing Overview

After concluding development of the full cyber physical prototype, testing must be carried out to ensure that it meets the goals that it was intended to, or at least reasonably show that it can do so with further refinement.

One important thing to note here is that the object sorter prototype is NOT optimized for speed. It was tuned entirely with accuracy in mind. With further tuning and refinement, some time can be saved. Areas of refinement include: speeding up the servo, reducing the cooldown, better model training, etc. I will not consider these future improvements as these are more refinements that can be done to enhance and optimize what is already there.

## Test Setup and Scenario

I took some assorted coins I had at home and mixed them up. This pile comprises of pennies, dimes, and quarters. My goal is to be able to use the model trained to be able to sort through this pile.

Throughout the testing I want to answer the following questions:

* How does the prototype compare to a human (myself) when both are binary sorting?
* How does the prototype compare to a human (myself) when the human sorts without limitations?

### Testing Methodology

Below is a list of rules and/or notes regarding the testing methodology:

* Repeatability is required on the most tests to use averages as a baseline rather than relying just on a human being consistent (as humans are consistently inconsistent).
* In general, I will be working at a moderate pace, not too slowly or quickly.
* When I am binary sorting, I will have all objects in a pile, and pull from them one at a time without looking at what I am grabbing from the pile. If the object I select matches what I am looking for, it goes in one pile, and if it does not, it goes into a 'discard' pile. Once the pass has been completed, the discard pile is poured to where the original pile was, and the process is repeated until all objects have been sorted. After sorting, then the objects are counted, written down, and added together to calculate statistics (minus throughput).
* Tests carried out that do not limit the human (myself) will only be ran once to get a picture of the differences rather than averaged.
* When using the prototype, any time spent between passes counts as part of passing, so it is important to ensure that the end user doesn't waste too much time during these downtimes.

### Testing Plan

A pile of coins was chosen to be the test pile. It is the same pile used across all tests (including manual and prototype testing). The pile consists of 15 Quarters, 15 Pennies, 26 Dimes, which yields a total of 56 objects sorted.

1. Human Sorting in Binary Fashion:
    * HSB-1: QUARTER-PENNY-DIME
    * HSB-2: DIME-QUARTER-PENNY
    * HSB-3: PENNY-DIME-QUARTER
    * HSB-A: Average of above 3 trials
2. Human Sorting (No Binary):
    * HSNB-1: Allowed to take one object at a time from big pile, but able to categorize into smaller sorted piles
    * HSNB-2: Allowed to see full pile and sort into smaller sorted piles
3. Prototype Sorting:
    * PS-1
    * PS-2: Recorded on Video (shown below)
    * PS-3: Used supplemental lighting to address any concerns about lighting
    * PS-A: Average of above 3 trials

Bonus Test: How long does it take to filter out the pennies from the whole pile?

1. Prototype (PF-1)
2. Manual (HF-1): Must pull from pile one at a time without looking, determine whether object selected is a penny, then act accordingly
3. Manual (HF-2): Pour entire pile out, then filter out all pennies and everything else

### Benchmarks

* 90% accuracy between actual objects and amount counted (through whichever sorting process was used)
* Object sorter must be faster than the times that a human (myself) sorting in a binary fashion

## Video

<div class="video-wrapper">
  <iframe 
    src="https://www.youtube.com/embed/uLqGxyvlCjE?mute=1&rel=0&playsinline=1"
    title="Trial PS-2"
    frameborder="0"
    allow="autoplay; encrypted-media"
    allowfullscreen>
  </iframe>
</div>

<div><br></div>

## Results

Table 1 - Speed Comparison

| Test ID | Time (MM:SS) | Throughput (objects/min) |
| ------------------ | ------------------ | ------------------ |
| HSB-1 | 05:36 | 10.00 |
| HSB-2 | 05:18 | 10.56 |
| HSB-3 | 05:29 | 10.21 |
| HSB-A | 05:28 | 10.24 |
| HSNB-1 | 02:41 | 20.87 |
| HSNB-2 | 1:41 | 33.27 |
| PS-1 | 05:01 | 11.16 |
| PS-2 | 04:26 | 12.63 |
| PS-3 | 04:18 | 13.02 |
| PS-A | 04:35 | 12.22 |

Findings:

* Human sorting in a non-binary fashion is BY FAR faster than anything done in a binary fashion.
* Binary sorting done by a human or by the prototype is comparable to one another, and overall, the prototype was about 16% quicker on average. Using HSB-A as a baseline, the prototype was capable of being around 21.3% quicker.
* There is potential to make the prototype faster, especially trimming down the time the servo rotates to either bin and back to the neutral position, however, it is best to minimize risk and ensure it works.

Table 2 - YOLO Accuracy Per Trial

| Test ID | Pennies Detected | Dimes Detected | Quarters Detected | Type of Error (ACTUAL OBJECT &rarr; DETECTED OBJECT) | Number of Errors | Accuracy | 
| ------------------ | ------------------ | ------------------ | ------------------ | ------------------ | ------------------ | ------------------ |
| HSB-1 | 13/15 | 26/26 | 17/15 | Penny &rarr; Dime, Penny &rarr; Quarter, Dime &rarr; Quarter | 3 | 94.6% |
| HSB-2 | 14/15 | 22/26 | 20/15 | Penny &rarr; Quarter, Dime &rarr; Quarter (x4) | 5 | 91.1% |
| HSB-3 | 16/15 | 27/26 | 13/15 | Quarter &rarr; Dime, Quarter &rarr; Penny, Dime &rarr; Quarter | 3 | 94.6% |

Findings:

* Pennies and Dimes are fairly strong, with quarters being the weak point
* Detection of quarters led to the most mistakes
* Lighting makes a noticable difference, but overall did not impact the results in the end. On one trial the errors were close to zero but I had to abandon that due to issues with the Arduino.
* Slight overcounting occured in Trial HSB-2, which is as a result of the object getting 'stuck' on the platform due to geometric limitations seen with FDM 3D Printing

Table 3 - Penny Extraction Benchmark

| Test ID | Time (MM:SS) | Throughput (objects/min) | Notes |
| ------------------ | ------------------ | ------------------ | ----------------------------------------- |
| PF-1 | 02:15 | 24.44 | Prototype was 100% accurate in separating pennies |
| HF-1 | 02:23 | 23.07 | Slower than the prototype |
| HF-2 | 00:43 | 76.74 | Humans benefit from global view, but really only applies in situations when global view CAN be utilized (e.g. seeing 1000 resistors for sorting would be tricky) |

## Remarks

Overall, the prototype was markedly faster in binary sorting situations compared against myself. However, humans have the benefit of a global view, which is something that isn't really possible with a YOLO Model. The overarching concept of a binary sorter inherently limited the scope of improvement by quite a bit. 

The speed of the object sorter can be improved in 2 main ways:

1. Improve the model being used. This could also entail swapping the version of YOLO used or trying out a different line of model all together.
2. Reduce the time spent waiting for the object to fall off the platform. If even 0.5 seconds can be saved here, across 50 objects, that would be a full 25 seconds saved, which can be seen as valuable time in a fast-paced environment.