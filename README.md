# Root-me CTF challenge writeup : Java - Custom gadget deserialization

## 1. What is Serialization and Deserialization  

### &emsp;Serialization
&emsp;&emsp;is the process of converting a Java object into a stream of bytes so that it can be stored (e.g., in a file or database) or transmitted (e.g., over 
&emsp;&emsp;a network). This allows complex data structures, like classes or objects, to be saved and later reconstructed.

### &emsp;Deserialization
&emsp;&emsp;It is the reverse process: it takes a stream of bytes and reconstructs the original Java object in memory.  
  
------
  
&emsp;In a secure context, serialization is useful for persisting application state or sending objects between systems. However, when deserialization 
&emsp;is done on untrusted data, it can become a serious security risk. If an attacker manages to send crafted serialized data, and if the application 
&emsp;blindly deserializes it without validation, it can lead to arbitrary code execution.


## 2. Gathering information and understanding the code

&emsp;Starting the challenge, we are welcomed by the following login page  

![Login_page](https://github.com/K8avid/CTF_Java-_Custom_gadget_deserialisation/blob/main/login_page.png)

&emsp;We can use Burp Suite to see if there is anything interesting in the HTTP request we send by entering the page or attempting to log in, and 
&emsp;we did find an interesting CSRF token:

![Login_page](https://github.com/K8avid/CTF_Java-_Custom_gadget_deserialisation/blob/main/post_request_csrf.png)

&emsp;And it does look like it is being encoded in base64, we can try to decode it and we get the following result:

![Login_page](https://github.com/K8avid/CTF_Java-_Custom_gadget_deserialisation/blob/main/csrf_decoded.png)

&emsp;It seems like we are chaining methods from an initial class called `UUID`, there does not seem to be more — let's dive into the provided   
&emsp;source code!

&emsp;Directed by our find, let's check the methods that are handling the `/login` route.  
&emsp;First of all, we can see that in the `HomeIndex` class, located in `src/main/java/com/rootme/serial`,  

![Login_page](https://github.com/K8avid/CTF_Java-_Custom_gadget_deserialisation/blob/main/get_login.png)  

&emsp;GET to the `/login` route creates a CSRF token and adds it to the model:

![Login_page](https://github.com/K8avid/CTF_Java-_Custom_gadget_deserialisation/blob/main/post_login.png)  

&emsp;The POST method to the `/login` route takes a CSRF token as a parameter and then deserializes it by calling `deserialize()` from the   
&emsp;`SerializationUtils` class.  
&emsp;After decoding it from base64, the `deserialize()` method itself calls `readObject()`, which will read and recreate the object:

![Login_page](https://github.com/K8avid/CTF_Java-_Custom_gadget_deserialisation/blob/main/deserialize.png)  

&emsp;The base64 decoder has nothing in particular, but if we take a closer look at `readObject()`, it calls a `run()` method that requires a boolean  
&emsp;`debug` field of the class `DebugHelper` it is in, to be `true`.  
&emsp;The `readObject()` method belongs to a `DebugHelper` class in the same folder, which means that the CSRF token we will pass should be an  
&emsp;object of that class.

&emsp;Here at the beginning of the `run` method:  

![Login_page](https://github.com/K8avid/CTF_Java-_Custom_gadget_deserialisation/blob/main/run.png)  

&emsp;If we can set the `debug` field to `true`, by reading the function we can see that it will start by decoding a base64 encoded command, then  
&emsp;execute it and store the result in the `output` field.  
&emsp;The following `toString()` method has been overridden and prints the output when calling the `print` method on a `DebugHelper` object:

![Login_page](https://github.com/K8avid/CTF_Java-_Custom_gadget_deserialisation/blob/main/toString.png)  

&emsp;At this point we can sense that, if we provide an ingeniously crafted CSRF token, we will most likely be able to execute remote code, so let's  
&emsp;try to serialize ourselves a `DebugHelper` type object with the right fields!

## 3. Creating the right payload  

&emsp;To craft the CSRF payload, we can create our own class in the folder we downloaded, use the different functions available and finally print the  
&emsp;result. Here is an example:

![Login_page](https://github.com/K8avid/CTF_Java-_Custom_gadget_deserialisation/blob/main/payload_gen.png)  

&emsp;After copy-pasting the payload to the CSRF field of the HTTP request in Burp Suite, you will get a response containing the desired flag:

![Login_page](https://github.com/K8avid/CTF_Java-_Custom_gadget_deserialisation/blob/main/final.png)

## Hope you enjoyed :)
