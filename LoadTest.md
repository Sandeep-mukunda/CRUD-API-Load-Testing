1. Make sure you have the latest Postman app https://www.postman.com/downloads/

<img width="379" height="357" alt="Screenshot 2025-07-15 at 5 01 34 PM" src="https://github.com/user-attachments/assets/3cbf468e-4a41-4448-bced-20375b6504b2" />

2. Under Collections, click “+”
<img width="389" height="431" alt="Screenshot 2025-07-19 at 10 29 07 AM" src="https://github.com/user-attachments/assets/31413338-4d08-44e3-9d4a-5fef25c78cc4" />


Give a name

Click “Add a request”
<img width="378" height="361" alt="Screenshot 2025-07-19 at 10 31 15 AM" src="https://github.com/user-attachments/assets/6165fda7-2028-4fc1-b8b9-c3f86ec38f99" />

IMPORTANT: Make sure to change “GET” to “POST”, and paste the API Gateway POST URL

Click “Body” then “raw” and paste:
<img width="828" height="206" alt="Screenshot 2025-07-15 at 5 06 35 PM" src="https://github.com/user-attachments/assets/8536587b-75a9-47f5-93a9-04adfd8252b2" />
<img width="853" height="444" alt="Screenshot 2025-07-19 at 10 38 58 AM" src="https://github.com/user-attachments/assets/2c1e9986-5de3-4f0b-9670-0662137da867" />
<img width="1440" height="725" alt="post" src="https://github.com/user-attachments/assets/c470ee39-1adc-4fb6-84c1-4c07a01435aa" />

MPORTANT: CLICK SAVE, ELSE IT WILL NOT SAVE!

<img width="843" height="385" alt="Screenshot 2025-07-15 at 5 12 37 PM" src="https://github.com/user-attachments/assets/fb38edcb-531a-4b01-8d32-f77a9792ba09" />

Validate: The request should change to POST, under the collection on left
<img width="1259" height="446" alt="Screenshot 2025-07-19 at 10 46 21 AM" src="https://github.com/user-attachments/assets/6ec12849-8f96-400c-be4e-dc4e5094d649" />
<img width="1268" height="557" alt="Screenshot 2025-07-19 at 10 47 05 AM" src="https://github.com/user-attachments/assets/c29f74d5-49d7-432c-92ba-eedd690b4dff" />


Click “Performance”, then select “Ramp up” under Load Profile, select “10” in Virtual users, and Test duration as 2 mins. Click Run!
<img width="901" height="728" alt="Screenshot 2025-07-19 at 10 49 48 AM" src="https://github.com/user-attachments/assets/65b2aadc-f2bb-4da3-b4c5-b82ee7d41cf6" />

Let it run for 2 mins, enjoy the show:
<img width="902" height="667" alt="Screenshot 2025-07-14 at 6 11 32 PM" src="https://github.com/user-attachments/assets/452c2234-59ee-4173-8612-78682abbeec7" />


After 2 mins is over and the run completed, click “...”, and download pdf. So, this is initial test, note the average response time in pdf. In my case it’s 389 ms, it can be different in your case


<img width="1032" height="648" alt="Screenshot 2025-07-14 at 5 36 13 PM" src="https://github.com/user-attachments/assets/71a3f865-ef56-4b74-992a-7e193c01a910" />
Now, let’s improve performance! Change the Lambda memory from 128 to 1024 MB under configuration, change timeout to 5 secs, save. So now  

<img width="670" height="372" alt="Screenshot 2025-07-19 at 10 57 43 AM" src="https://github.com/user-attachments/assets/308498eb-ac14-472d-806e-8397d07747db" />

As we increase Lambda memory, it gets more vCPU (mapping below)
<img width="625" height="420" alt="Screenshot 2025-07-19 at 10 58 47 AM" src="https://github.com/user-attachments/assets/7b802a7a-9836-4471-9e84-81a2dcb324b5" />

Now that our Lambda has 0.5 vCPU instead of 0.0625, it should perform faster. Let’s test it out

Go back to postman, and click “Run Again”
<img width="904" height="130" alt="Screenshot 2025-07-19 at 11 02 37 AM" src="https://github.com/user-attachments/assets/75f4399c-c7c3-43d0-8451-1ea71a5f5b2a" />


Export pdf again. And this time, response time came down because Lambda has more memory and vCPU


<img width="1038" height="443" alt="Screenshot 2025-07-14 at 5 28 02 PM" src="https://github.com/user-attachments/assets/7e0d1bdb-32a2-4d27-b82d-6247b91b5c1e" />


Customers run Lambda Power Tuning 
