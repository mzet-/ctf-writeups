## Vulnerability:

Price of the HDTV is sent as POST data as a hidden form field ('Price') so it is user controlled.

## Exploitation:
I used 'Live HTTP headers' firefox add-on to modify price field to 0.99 and purchased product for new price :)

## Mitigation:
Price value should be stored on server side. User input in POST (and of course in GET) requests can't be trusted because those can be trivially modified.

In general user input can't be trusted and need to be validated on the backend before use. 
