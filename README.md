# Root-me CTF challenge : Java - Custom gadget deserialisation


Starting the challenge, we are welcomed by the following login page  

![Login_page](https://github.com/K8avid/CTF_Java-_Custom_gadget_deserialisation/blob/main/login_page.png)

We can use Burp Suite to see if there is anything interesting in the HTTP request we send by entering the page or attempting to log in, and we did find an interesting CSRF token 

![Login_page](https://github.com/K8avid/CTF_Java-_Custom_gadget_deserialisation/blob/main/post_request_csrf.png)

And it does look like it is being encoded in base64, we can try to decode it and we get the following result :


