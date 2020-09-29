### Long Password DoS Attack

- As the value of password is hashed and then stored in Databases. If there is no limit on the length of the Password, it can lead to consumption of resources for Hashing the Long Password.

#### How to test?

- Use a Password of length around 150-200 words to check the presense of Length Restriction
- If there is no Restriction, Choose a longer password and keep a eye on Response Time
- Check if the Application Crashes for few seconds 

#### Where to test?

- Registration Password Field is usually restricted but the Length of Password on the Forgot Password Page and the Change Password (As Authenticated User) Functionality is usually missing.

#### Resources/References:

- https://www.acunetix.com/vulnerabilities/web/long-password-denial-of-service/
- https://hackerone.com/reports/738569
- https://hackerone.com/reports/167351

Author - [The Infosec Guy](twitter.com/g0t_rOoT_)
