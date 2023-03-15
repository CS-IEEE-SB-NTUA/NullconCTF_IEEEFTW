
# Challenge Description
HTTP requests and libraries are hard. Sometimes they do not behave as expected, which might lead to vulnerabilities.

# Solution
We are given the Source Code of a web service.
We have 2 python files, one for the frontent (app.py) and one for the backend (backend.py)

Looking at these lines  :
app.py:
```python
r = requests.Request("GET", "http://backend:8080/whoami", cookies=cookies, headers=headers)
```
backend.py:
```python
@app.route('/whoami')
```

This means that the frontend (the page we are in) sends a get request to the backend including the cookies of the page.

In backen.py we also see:
```python
	role = request.cookies.get('role','guest')
	really = request.cookies.get('really', 'no')
	if role == 'admin':
		if really == 'yes':
			resp = 'Admin: ' + os.environ['FLAG']
		else:
			resp = 'Guest: Nope'
	else:
		resp = 'Guest: Nope'
	return Response(resp, mimetype='text/plain')

```

this means that the backends checks for the cookies named `role` and `really` and returns the flag if they have the values `admin` and `really` reppectivelly.

In our browser we open the Developer Tools and in Storage we create 2 new cookies : 
`role : admin` and `really : yes`. This way after reloading the page we get this:

``` text
Usage: Look at the code ;-)

Overwriting cookies with default value! This must be secure!
Prepared request cookies are: [('PHPSESSID', '137656440e0d78fa17a6c5cafa536c3b'), ('role', 'guest'), ('really', 'yes')]
Sending request...
Request cookies are: [('PHPSESSID', '137656440e0d78fa17a6c5cafa536c3b'), ('role', 'guest'), ('really', 'yes')]

Someone's drunk oO

Response is: Admin: ENO{R3Qu3sts_4r3_s0m3T1m3s_we1rd_dont_get_confused}
```
