# Attacking the STM32F030F4P6 microcontroller attached to the ChipWhisperer Nano

<img width="2048" height="1536" alt="image" src="https://github.com/user-attachments/assets/d8cf8d73-b3ef-4a9b-a5a2-a439a34a98b9" />


I will be using a Jupyter Notebook VM to execute this SPA attack on. The ChipWhisperer Nano provides a helpful utility for this function, as I do not have access to an oscilloscope. Despite its lower sample rate of 20MS/s, it will suffice for this simple attack.


**Setup**

<img width="582" height="297" alt="image" src="https://github.com/user-attachments/assets/14dbcc67-31a4-443a-a53e-94ecaac5c437" />

First we will navigate to where our test firmware lies, and begin to confiure our target. Now we we are ready to flash the firmware onto our onboard STM32F0. We are going to be flashing the basic-passwdcheck, which I have included in this repository for reference during this exercise. The actual password is hardcoded in the file, but the functions of the program are vulnerable to simple power analysis.

<img width="1350" height="211" alt="image" src="https://github.com/user-attachments/assets/d9d2c15c-0e55-4c54-849f-3088335cf8ee" />


<img width="757" height="439" alt="image" src="https://github.com/user-attachments/assets/70a0bf78-6b36-4502-93ed-3b0e7eca640f" />

We have seen the boot messages from the device, it seems it is secure and requires a password for access. Now it is time to capture power traces from the device and start the analysis process.

<img width="684" height="574" alt="image" src="https://github.com/user-attachments/assets/f219922f-323d-4a18-8e00-1214ffd03995" />

We can plot a trace of a random password, in this case: hunter2. Let's see what the power trace graph looks like. 

<img width="1405" height="764" alt="image" src="https://github.com/user-attachments/assets/71098f2d-a165-4cd4-bcf8-4647955a3a3e" />

It looks like it begins to become repititve rather quickly. We know that the boot password is 5 characters long and that 'h' is the first letter of the password (to speed up the lab process, you could work without this information). Let's plot our traces on a graph using matplotlib comparing 'h' to something else, in this case, '0':

<img width="1063" height="761" alt="image" src="https://github.com/user-attachments/assets/8043a6a9-f00c-46d3-8b94-c5f7b2c86d1c" />

We can see a significant difference in the tracings from one boot sequence to another based on the differing password input, which tells us that this firmware is likely vulnerable to a simple power analysis attack. In this case, we know 'h' was the correct character, and we could test this by comparing '0' to another incorrect character.

<img width="1351" height="763" alt="image" src="https://github.com/user-attachments/assets/380c0885-c666-4aaa-88c4-6fa1fb88f900" />

In this case, I changed the password to check for 'i' and '0' instead of 'h' and '0', and as we can see, the power traces are practically identical, proving both characters are incorrect for the first character of the password. Now assuming we did not know what the first character should be, we can run a script to compare a null password, which is handeled the same as an incorrect password in this case, and compare it to each accepted character to find which one differs. 

<img width="992" height="753" alt="image" src="https://github.com/user-attachments/assets/0fc2943d-602b-4f46-a027-c44fd792a26a" />

As you can see, one of them is significantly different than the rest. So, let's use some numpy math functions to measure the actual differences so we can get a real measure instead of just eyeballing it. We can use np.sum(np.abs(DIFF)) to take the absolute value of each sample and add them all up. Now we will have a number representing the SAD (sum of absolute difference of all sample values), and one should be largely different than the rest.

<img width="674" height="643" alt="image" src="https://github.com/user-attachments/assets/bd64fbfa-6020-4b34-993a-d6991d6d8c26" />

As you can see, 'h' has a value of 489, while most other values barely get past 20 (not all values can be seen in the figure but assume they are similar). 

____________________________________________________________________________________________________________________________________________________________________

Now, let's run the full attack. We know that the differences for incorrect password attempts never made it past 22, and that our correct password reached almost 500, so we simply need to write a program that attacks each character in sequence and measures their difference to see if the character was correct or incorrect. 

<img width="1370" height="467" alt="image" src="https://github.com/user-attachments/assets/01f3f043-bcfb-4bf3-aaaa-8c406519c61a" />

This program measures the trace, sees if its sum of absolute difference is greater than 40 (a safe reference number), and then continues across the sequence once it has found the correct character, going until 5 characters (our max password length). As seen above, it has found the password: "h0px3". We have successfully extracted the password and hacked the target device!
