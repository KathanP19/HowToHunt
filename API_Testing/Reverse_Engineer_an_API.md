# Reverse Engineer an API
<img src="https://pasteboard.co/n2094TfDB8qt.png">

## Tools to use
1. FoxyProxy
2. mitmweb
3. mitmproxy2swagger
4. https://editor.swagger.io/
5. Postman

## Steps to Reproduce
1. **Foxyproxy:** Turn on 8080 port using Foxy Proxy.(Label it anything you want)
2. **mitmweb:** Run `sudo mitmweb` and then go to mitm.it and install & import the certificate.
3. **Explore Website w/ API's functionalities:** Go to the website w/ api that you want to gather the API endpoints from and explore it's functionalities. <br>The mitmweb tool will capture it,
afterwards you can download the captures as a flow file in mitmweb by clicking on file -> save all.
4. **mitmproxy2swagger:** Here we run `sudo mitmproxy2swagger -i flows -o spec.yml -p <website api> -f flow`. This will turn flows file to a yml file. Afterwards you need to remove the ignore: in the spec.yml and run
`sudo mitmproxy2swagger -i flows -o spec.yml -p <website api> -f flow --examples`, --examples is added to enhance the documentation of the api endpoints.
5. **https://editor.swagger.io/:** Now you can import the clean spec.yml file and visualize the different endpoints.
6. **Postman:** You can also import the spec.yml in postman which will produce a well organized collection.
