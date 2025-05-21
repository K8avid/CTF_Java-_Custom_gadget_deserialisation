# Root-me CTF challenge : Java - Custom gadget deserialisation


Starting the challenge, we are welcomed by the following login page  

![Login_page](https://github.com/K8avid/CTF_Java-_Custom_gadget_deserialisation/blob/main/login_page.png)

We can use Burp Suite to see if there is anything interesting in the HTTP request we send by entering the page or attempting to log in, and we did find an interesting CSRF token 

![Login_page](https://github.com/K8avid/CTF_Java-_Custom_gadget_deserialisation/blob/main/post_request_csrf.png)

And it does look like it is being encoded in base64, we can try to decode it and we get the following result :

![Login_page](https://github.com/K8avid/CTF_Java-_Custom_gadget_deserialisation/blob/main/csrf_decoded.png)

It seems like we are chaining methodes from an initial class called UUID, there does not seem to be more let's dive into the provided source code !  

First of all, we can see that in the HomeIndex class, located in srs/main/,  

![Login_page](https://github.com/K8avid/CTF_Java-_Custom_gadget_deserialisation/blob/main/get_login.png)  
GET to the /login route creates a CSRF token and adds it to the model  


![Login_page](https://github.com/K8avid/CTF_Java-_Custom_gadget_deserialisation/blob/main/post_login.png)  

the POST methode to the /login route takes a CSRF token as a parameter and then deserializes it by calling deserialize() from the SerializationUtils class
After decoding it from base64 the deserialize methode itself calls readObject(), which will read and recreate the object,  




