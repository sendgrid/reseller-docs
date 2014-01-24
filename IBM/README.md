### Managing Services

To create and bind a new SendGrid service, see [Managing Services from the Command Line](https://ace.ng.bluemix.net/docs/QuickStart.jsp).

### Creating a SendGrid Service

SendGrid can be provisioned with the following command:

```
$ cf create-service sendgrid [service-name] [plan-level]
```

The service name can be anything you want and the plan level is one of these options: free.

OR

```
$ cf create-service
```

cf prompts you to choose one of the available service offerings, and then ask you for a unique instance name and the plan for the service. It will present you with a random name but you can change it if you want.

### Binding Your SendGrid Service

After creating a new SendGrid service, you can use the _cf services_ command to list all available service instances that you have created:

```
$ cf services
```

Bind your SendGrid service to your app, using the following command:

```
$ cf bind-service [service-name] [app-name]
```

The service name should match the one you provisioned above and the app name should be an existing IBM SmartCloud app.
Once SendGrid has been added a username and password will be available. These are the credentials you use to access the newly provisioned SendGrid service instance.

OR

```
$ cf bind-service
```

You need to select one of the applications and one of the services from the list. Once the binding action succeed, cf will return a message to you.

### Using SendGrid within your Application

Once a SendGrid service instance has been bound to your application, the VCAP_SERVICES Environment Variable will be automatically updated to include your credentials. The section of `VCAP_SERVICES` that pertains to SendGrid will look like this:

```
{
   "SendGrid-1.0.0": [
      {
         "name": "SendGrid-paxza",
         "label": "SendGrid-1.0.0",
         "credentials": {
            "SENDGRID_USERNAME": "edc4c53c286825a2e600a28f9efdc1cd",
            "SENDGRID_PASSWORD": "Z2qwbS1EB1",
            "SENDGRID_SMTP_HOST": "smtp.sendgrid.net"
         }
      }
   ]
}
```

As you can see, the VCAP_SERVICE environment variable includes the following items:

*   SENDGRID_USERNAME: The username of the SendGrid account
*   SENDGRID_PASSWORD: The password of the SendGrid account
*   SENDGRID_SMTP_HOST: The host of the SendGrid smtp host

You can use the following code snippet to obtain the service credential information and connect to the SendGrid instance:

#### 1. Ruby on Rails

Add this to the config/envirorment.rb:

```Ruby
credentials = host = username = password = ''
if !ENV['VCAP_SERVICES'].blank?
  JSON.parse(ENV['VCAP_SERVICES']).each do |k,v|
    if !k.scan("SendGrid").blank?
      credentials = v.first.select {|k1,v1| k1 == "credentials"}["credentials"]
      host = credentials["hostname"]
      username = credentials["username"]
      password = credentials["password"]
    end
  end
end
 
ActionMailer::Base.smtp_settings = {
  :address        => host,
  :port           => '587',
  :authentication => :plain,
  :user_name      => username,
  :password       => password,
  :domain         => 'sendgrid.example.com',
  :enable_starttls_auto => true
}
```

After which you can send all your email using the ActionMailer class through SendGrid.

#### 2. Node.js

To use SendGrid with Node.js you first need to add our library to your dependencies in your package.json file:

```
"dependencies": {
 ...
 "sendgrid": "1.0.0-rc.1.0",
 ...
}
```  

After which you can get the credentials from VCAP_SERVICES and use our library to send emails:

```JavaScript
if (process.env.VCAP_SERVICES) {
    var env = JSON.parse(process.env.VCAP_SERVICES);
    var credentials = env['SendGrid-1.0.0'][0].credentials;
} else {
    var credentials = {
        "SENDGRID_SMTP_HOST": "smtp.sendgrid.net",
        "SENDGRID_USERNAME" : "user1",
        "SENDGRID_PASSWORD" : "secret"
    }
};
 
var sendgrid  = require('sendgrid')(credentials.SENDGRID_USERNAME, credentials.SENDGRID_PASSWORD);
sendgrid.send({
  to:       'example@example.com',
  from:     'other@example.com',
  subject:  'Hello World',
  text:     'My first email through SendGrid.'
}, function(err, json) {
  if (err) { return console.error(err); }
  console.log(json);
});
```

The code listed above firstly checks whether the VCAP_SERVICES environment variable exists

### 3. Java

To use SendGrid with Java you first need to add [our library](https://github.com/sendgrid/sendgrid-java#via-copypaste) in your project.

After which you can get the credentials from VCAP_SERVICES and use our library to send emails:

```Java
JSONObject sendgridJson = new JSONObject(System.getenv("VCAP_SERVICES"));
JSONObject sendgridCredentials = sendgridJson.getJSONArray("SendGrid-1.0.0")
  .getJSONObject(0).getJSONObject("credentials");
String username = sendgridCredentials.getString("SENDGRID_USERNAME");
String password = sendgridCredentials.getString("SENDGRID_PASSWORD");
         
SendGrid sendgrid = new SendGrid(username, password);
sendgrid.addTo("example@example.com");
sendgrid.setFrom("other@example.com");
sendgrid.setSubject("Hello World");
sendgrid.setText("My first email through SendGrid / IBM");
sendgrid.send();
```

### Pushing your application to SmartCloud

After finished the coding, you can deploy your application to Bluemix environment for verification. To deploy an application, you need to enter into the root directory of the application and use the following command:

```
$ cf push
```

or for node.js:

```
$ cf push --command 'node index.js'
```

### <span style="color: rgb(0,0,0);">Unbinding or deleting a SendGrid service instance</span>

*   To unbind a service instance from an application, you can use the _`cf unbind-service`_ command.
*   To delete a service instance, use the _`cf delete-service`_ command.

### Sample Application

You can use the following sample applications:

*   Ruby on Rails: [https://github.com/cloudfoundry-samples/sendgrid-cloudfoundry-rails](https://github.com/cloudfoundry-samples/sendgrid-cloudfoundry-rails)
*   Node.js: [https://github.com/scottmotte/sendgrid-nodejs-example](https://github.com/scottmotte/sendgrid-nodejs-example)

These applications demonstrate how to connect to our services and insert or retrieve data.

### Related Links

[SendGrid Official Document](http://sendgrid.com/docs/index.html)

### Contact

If additional support is needed, you can contact [support](https://sendgrid.zendesk.com/hc/en-us).
