## n8n-Error-Handling

#### Error Handling nam e ekta google sheets toiry koro--->field thakbe: Name, Email---> value daw: Wasim, mdwasimu02@gmail.com

#### Open nodes panel click koro--->search koro: google sheets--->click koro: On raw added--->value set koro--->Fetch Test Event click koro

#### supabase dashboard e jaw--->Table Editor e jaw--->click koro: New Table--->Name daw: Errors--->Columns e value set koro & save koro--->
![](https://imgur.com/WpbRShr.png)

#### Google Sheets Trigger er + sign click koro--->search & click: supabase--->click: Create a row--->drag & drop kore value set koro: Add field click kore.--->Execute step click koro.

#### click koro: Create a row er +sign--->search & click: if --->Conditions e is equal to select koro--->value daw: {{ $json.error }}

#### if er false e +sign click koro--->search & click: gmail--->click: send a message--->Send a message er nam change kore Success daw & value set koro.
#### if er true e +sign click koro--->search & click: gmail--->click: send a message--->Send a message er nam change kore Failed daw & value set koro.

#### google sheets e value add kore execute workflow te click koro--->jodi same email ekadik thake tahole error hobe & failed e message jabe ar jodi unique email hoi tahole success e message jabe.
