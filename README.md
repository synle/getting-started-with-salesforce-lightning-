# Getting Started With Salesforce Lightning
===========================================


## What is Force.com?
- A collection of tools for developers provided by Salesforce, can be used to build epic apps in the Salesforce ecosystem. Things include Apex (a variation of Java language) which can be used to write server side logics or talking to Salesforce API using SOSL or SOQL.



## Tools you need to get started.
- Syncing your code from force.com: using either [Mavenmates](http://mavensmate.com/) or [Force.com cli](https://force-cli.heroku.com/). They are all decent tools, works for all platforms: Mac, Linux and Windows. Force CLI is a great tool if you plan to do all your works from the command line. Mavenmates has great integration for Sublime, Atom and Visual Studio Code. For force cli, there is a plugin for Sublime called [Sublime Lightning](https://github.com/dcarroll/sublime-lightning) which provides great integration with force cli

- Your editor of choice: can be Sublime, Atom, Visual Studio code (your call) or even VIM. But with Vim, things will get tricky with the integration (you can use force cli to push your changes to force.com).

## Getting started
- Sign up for a Salesforce Developer Org (SDO) [here](https://developer.salesforce.com/signup)

- Create your component of choice using Developer Console
- Set up a domain for your org, go to My Setup > My Domains, then pick a domain.

## Ways to make calls to your API ( non Salesforce )
### Using Apex Controller
You can use Apex Controller to make call to your API. You need to do the following. Visit this link from Salesforce for [information about making call out from Apex](https://developer.salesforce.com/page/Apex_Web_Services_and_Callouts)
1. You need to allow Remote Site in Salesforce to allow calls to non-salesforce domain
2. Below is a snippet of Apex code which can be used to make call out to your app.
```
HttpRequest req = new HttpRequest();

//Set HTTPRequest Method
req.setMethod('PUT');

//Set HTTPRequest header properties
req.setHeader('content-type', 'image/gif');
req.setHeader('Content-Length','1024');
req.setHeader('Host','s3.amazonaws.com');
req.setHeader('Connection','keep-alive');
req.setEndpoint( this.serviceEndPoint + this.bucket +'/' + this.key);
req.setHeader('Date',getDateString());

//Set the HTTPRequest body
req.setBody(body);

Http http = new Http();

try {
  //Execute web service call here
  HTTPResponse res = http.send(req);

  //Helpful debug messages
  System.debug(res.toString());
  System.debug('STATUS:'+res.getStatus());
  System.debug('STATUS_CODE:'+res.getStatusCode());

} catch(System.CalloutException e) {
  //Exception handling goes here....
}
```

### force cli commands
#### Login with a brand new account
```
myHost='login.salesforce.com'
force login -i=$myHost

myHost='mobile1.t.salesforce.com'
force login -i=$myHost -u=$myUsername -p=$myPassword
```

#### Guided set up script
```
if [ ! -f /usr/local/bin/force ]
then
  pushd /usr/local/bin
  sudo curl -O https://s3-us-west-2.amazonaws.com/force-cli/heroku/force/v0.22.67/darwin-amd64/force
  sudo chmod +x /usr/local/bin/force
  popd
  echo 'Please allow force cli access to your organization:'
  sleep 3
fi

echo "==="

echo -n "Use Mobile1 Environment (y/n)? "
read answer
if echo "$answer" | grep -iq "^y" ;then
  myHost='mobile1.t.salesforce.com'

  echo 'Salesforce Username:'
  read myUsername

  echo 'Salesforce Password:'
  read -s myPassword

  echo "Logging in $myHost as $myUsername"

  force login -i=$myHost -u=$myUsername -p=$myPassword

else
  echo 'login to public salesforce.com'
  myHost='login.salesforce.com'
  force login -i=$myHost
fi
```


Activate different already logged in profile
```
force active -a email@gmail.com
```


#### Pull new code from upstream force.com
```
force fetch -t AuraDefinitionBundle
force fetch -t CorsWhitelistOrigin
force fetch -t RemoteSiteSetting
force fetch -t StaticResource
```

#### Push code upstream force.com
```
force push -r -t AuraDefinitionBundle
force push -r -t CorsWhitelistOrigin
force push -r -t RemoteSiteSetting
force push -r -t StaticResource
```

#### Deploy code to new org
`force import` imports code from meta data folder, this following snippet will allow that
```
echo "porting code to new org from cli..."
metadataDir=metadata
rm -rf $metadataDir;
cp -rf src $metadataDir;
force import;
rm -rf $metadataDir;
```


#### Sample Force CLI Package.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<Package xmlns="http://soap.sforce.com/2006/04/metadata">
  <types>
    <members>*</members>
    <name>AuraDefinitionBundle</name>
  </types>
  <types>
    <members>*</members>
    <name>CorsWhitelistOrigin</name>
  </types>
  <types>
    <members>*</members>
    <name>RemoteSiteSetting</name>
  </types>
  <types>
    <members>*</members>
    <name>StaticResource</name>
  </types>
  <types>
    <members>*</members>
    <name>ApexClass</name>
  </types>
  <version>37.0</version>
</Package>
```
### Sample code to pull/push code using Force CLI
Please note that at the time of this writing, Force CLI doesn's support CSP (Content Security Policy) Meta Data
```
# to pull
force fetch -t CorsWhitelistOrigin
force fetch -t StaticResource
force fetch -t RemoteSiteSetting
force fetch -t ApexClass
force fetch -t AuraDefinitionBundle

# to push
force push -r -t StaticResource
force push -r -t RemoteSiteSetting
force push -r -t CorsWhitelistOrigin
force push -r -t ApexClass
force push -r -t AuraDefinitionBundle
```


### Making pure Ajax calls (on the client side with Salesforce Lightning)
You can use either XMLHttpRequest in JS or using the fetch API, whatever that fits your need.

Things you need to enable to make this work.
1. Allow CORs to your domain: it's a setting in Salesforce
2. Allow CSPs (Content Security Policy) to your domain: it's a setting in Salesforce.
3. Make sure your web app has a OPTION method which responded with a proper header for `Access-Control-Allow-Origin`, this also applies to the API call itself (/fetchYourData) has `Access-Control-Allow-Origin`. A header value of `*.force.com` is likely to be enough for your needs.
4. The code to make the ajax call as usual.

#### Sample Ajax calls with XMLHttpRequest
```
function sendAjaxRequest(method, url, formData, option, isSynchronous) {
    return new Promise(function(resolve, reject){
        var xhr = new XMLHttpRequest();

        // on ready params
        xhr.onreadystatechange = function onreadystatechange() {
            if (this.readyState === 4) {
                var responseCode = +this.status;
                var errorMessage = xhr.responseText || xhr.statusText || xhr.status;
                
                try{
                  if (responseCode >= 200 && responseCode <= 399) {
                    // success
                    return resolve( JSON.parse( xhr.responseText ) );
                  }
                } catch(e){}

                // error, all failed, then reject the promise
                var errorMessage = xhr.responseText || xhr.statusText || xhr.status;
                reject( errorMessage );
            }
        };


        // prepping get/post data
        var formDataToSend;
        if (formData) {
            switch (method) {
                case 'GET':
                    url = url + getEncodedQueryStringByParams(formData);
                    break;
                case 'POST':
                default:
                    formDataToSend = JSON.stringify(formData);
                    break;
            }
        }

        xhr.withCredentials = true;
        xhr.open(method, url, isSynchronous);

        // setting headers
        xhr.setRequestHeader("Content-Type", "application/json;charset=UTF-8");

        xhr.send( formDataToSend );
    }).catch(function(e){
        console.error('Network Call Failed: ', e)
        return Promise.reject(e);
    });
};


// send apex action
function sendApexRequest(cmp, methodName, params){
    return new Promise(function(resolve, reject){
        var action = cmp.get(methodName);// methodName is of form "c.getUserSetting"

        if (params) {
            action.setParams(params);
        }

        action.setCallback(this, function(response) {
            switch(response.getState()){
                case 'SUCCESS':
                    return resolve(response.getReturnValue());
                case 'ERROR':
                    return response.getError(response.getError());
            }
        });

        $A.enqueueAction(action);
    }).catch( function(e){
        console.error('APEX Call Failed', e);
    });
}




// inspired by http://stackoverflow.com/questions/3308846/serialize-object-to-query-string-in-javascript-jquery
// will return ?key=value...
function getEncodedQueryStringByParams(json){
    json = json || {};
    var jsonKeys = Object.keys(json);

    if (jsonKeys.length === 0) {
        return '';
    }

    return '?' +
        jsonKeys
            .reduce(function(res, key) {
                var dataToProcess = [json[key]];
                if(Array.isArray(json[key])){
                    // if is an array (array keys)
                    // key[]=value
                    dataToProcess = json[key];
                }

                // combine result
                return res.concat(
                    dataToProcess.filter(function(val){
                            return val !== null && val !== undefined;
                        })
                        .map(function(val){
                            return encodeURIComponent(key) + '=' + encodeURIComponent(val)
                        })
                );
            }, [])
            .join('&');
}
```

### Notes related to static bundle
#### Bundling your static resource
You can use your weapon of choice to bundle your static asset, I used `browserify` cli to achieve my goal, it's working great for me.
```
browserify src/index.js -o src/staticresources/MyAppBundle.resource
```


#### Including the bundle in your cmp or app
```
<ltng:require scripts="{!$Resource.MyAppBundle}"
  afterScriptsLoaded="{!c.onInit}" />
```



### Apex Survival
#### Aura Friendly Apex Class Properties
```
  @AuraEnabled
  public String userId  { get; protected set; }
```

#### Custom Settings
```
  my__Setting__c.getInstance(instanceName);
```

#### Unit test
Unit tests generally are not hooked up, if you need to access SFDC in your tests, attach this annotation `@isTest(SeeAllData=true)`
```
public class MyTest {
  @isTest(SeeAllData=true)
  static void validateHelloWorld() {
    System.debug('hello world');

    // assertion
    System.assertEquals(1, 1);
  }
}
```

### Package and Deploy
1. Go to Setup > Package Manager
2. Create a namespace for your app.
3. Create a bundle
4. Add required components that makes up your components
5. Upload it
6. Distribute the Installation link.


### Salesforce Lightning Tips and Tricks
#### Navgiate to SObject given we know the record id.
[navigateToSObject Doc](https://developer.salesforce.com/docs/atlas.en-us.lightning.meta/lightning/ref_force_navigateToSObject.htm)
```
createRecord : function (component, event, helper) {
    var navEvt = $A.get("e.force:navigateToSObject");
    navEvt.setParams({
      "recordId": "00QB0000000ybNX",
      "slideDevName": "related"
    });
    navEvt.fire();
}
```


#### Routing with e.force:navigateToComponent
[navigateToComponent Doc](https://developer.salesforce.com/docs/atlas.en-us.lightning.meta/lightning/ref_force_navigateToComponent.htm)
```
navigateToMyComponent : function(component, event, helper) {
    var evt = $A.get("e.force:navigateToComponent");
    evt.setParams({
        componentDef : "c:myComponent",
        componentAttributes: {
            contactName : component.get("v.contact.Name")
        }
    });
    evt.fire();
}
```


#### Navigate to Create Object with createRecord
[createRecord Doc](https://developer.salesforce.com/docs/atlas.en-us.lightning.meta/lightning/ref_force_navigateToComponent.htm)
```
createRecord : function (component, event, helper) {
    var createRecordEvent = $A.get("e.force:createRecord");
    createRecordEvent.setParams({
        "entityApiName": "Contact"
    });
    createRecordEvent.fire();
}
```
