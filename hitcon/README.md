# Angry Seam (500 pts)

![N|Solid](login.png)


The first page we can see is a login page. Hanging around a little, we can list a few interesting pages :
  - The css.seam page take a filename as parameter.
  - The logout button take a parameter that corresponds to "view:object.method" format.
  - The report.seam page permits to send a URL to the administrator, but only URL that is intern to the application.


With the css.seam page, we can list directories and files at the root of the application :

http://52.198.197.227:8080/angryseam/css.seam?location=../../../

![N|Solid](directorylisting.png)


Reading the WEB-INF/pages.xml, we can view that there is a flag.xhtml, and we can deduce there is a flag.seam too. However, we saw that we must be admin to access it. 

http://52.198.197.227:8080/angryseam/css.seam?location=../../../WEB-INF/pages.xml

![N|Solid](pagesxml.png)

The first thing I tried, with little hope, was to play with the session. **Logged in as lambda user**, I tried to go on register.seam and register with "admin" username.

![N|Solid](registerAdmin.png)

Error message, the user is existing. But... it changed my session, and I'm logged in as admin now! I can browse the flag.seam as admin, and tada!, the flag.

![N|Solid](flag.png)

I'm pretty sure this wasn't the intended way to solve this. But... I think it was the easiest way to do it! :)

_Blaklis from Fourchette-Bombe_