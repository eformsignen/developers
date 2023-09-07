--------------------------
Using eformsign API
--------------------------

The API provided by eformsign allows customers to call and use eformsign features in their system/service.



Getting started 
====================


You need the following information to use eformsign API.

- Company ID and Document ID
- API Key and Private Key
- Generating a signature

.. caution:: 
   
   There is a **30 second time limit** for generating a signature. You must create a signature and get token issued within 30 seconds. 



Where to find Company ID and Document ID
-----------------------------------------------------

To use eformsign API, you need to know the company ID and the document ID that you want to lookup. 

Log in to eformsign service and find the company ID and the document ID.

.. note:: 

   You can view the company ID in Manage company > Company profile > Basic information.

   |image1|

   Go to the document box and click the document icon (|image2|) at the top right corner and add Document ID column. Then you can view the document ID in the document list. 

   |image3|






.. _apikey:

Getting an API key and private key
----------------------------------------

1. Log in to eformsign as the company administrator and then go to **[Integration] > [API / Webhook]** in the sidebar menu.

.. image:: resources/apikey1.PNG
    :width: 700
    :alt: Integration > API/Webhook menu


2. Select the **[Manage API key]** tab and then click the **Create an API key** button.

.. image:: resources/apikey2.PNG
    :width: 700
    :alt: Create an API key button


3. Enter the alias and application name in the **Create an API key** pop-up.

.. image:: resources/apikey3.PNG
    :width: 300
    :alt: Create an API key pop-up


4. Select the **Authentication type** and click **Save**.

.. note:: 
    
   You can select from one of three authentication types: **Bearer token, Basic authentication, and eformsign signature**. 

   - **Bearer Token**\ : Uses a preset value for authentication.

    .. image:: resources/apikeyauth1.PNG
        :width: 300
        :alt: API key pop-up 1


    Select **Bearer token** as the authentication type and enter the value to be used as the token value under **Value** and click **Save**.  `When getting an access token <https://app.swaggerhub.com/apis-docs/eformsign_api.en/eformsign_API_2.0/2.0#/token/post-api_auth-access_token>`_, enter the token value in the format of the **Bearer token value** in the request header, eformsign_signature. Refer to the following example.

    .. code:: Javascript

        curl --location --request POST 'https://service.eformsign.com/v2.0/api_auth/access_token' \
        --header 'eformsign_signature: Bearer {token value}' \
        --header 'Content-Type: application/json' \
        --header 'Authorization: Bearer {base64 encoded api key }' \
        --data-raw '{
         "execution_time":{timestamp in ms},
         "member_id": {eformsign account}
        }'


   - **Basic Authentication**\ : Uses an ID and password for authentication. 

    .. image:: resources/apikeyauth2.PNG
        :width: 300
        :alt: API key pop-up 2



    Select **Basic authentication** as the authentication type and enter the ID and password, then click **Save**.  `When getting an access token <https://app.swaggerhub.com/apis-docs/eformsign_api.en/eformsign_API_2.0/2.0#/token/post-api_auth-access_token>`_, enter the Base64-encoded value in the format of **ID:Password** in the request header, eformsign_signature. Refer to the following example. 

    .. code:: Javascript

        curl --location --request POST 'https://service.eformsign.com/v2.0/api_auth/access_token' \
        --header 'eformsign_signature: Basic {base64 encoded "id:password"}' \
        --header 'Content-Type: application/json' \
        --header 'Authorization: Bearer {base64 encoded api key }' \
        --data-raw '{
         "execution_time":{timestamp in ms},
         "member_id": {eformsign account}
        }'


   - **eformsign Signature**\ : Uses the eformsign Signature value for authentication. Select the authentication type to **eformsign signature** and click **Save**. Refer to `Generating a signature <#eformsign-signature>`_ on how to sign a signature with eformsign Signature.




4. Select the **View key** button in the list of keys and view the API key and private key.

.. image:: resources/apikey4.PNG
    :width: 700
    :alt: View API key

.. image:: resources/apikey5.PNG
    :width: 700
    :alt: View private key



.. note:: **Editing an API key**

    You can change the alias and application name by clicking the **Edit** button in the API key list. You can also disable/enable the key by clicking the **Status** toggle button.

.. note:: **Deleting an API key**

    You can delete an API key by clicking the **Delete** button in the API key list.


.. _eformsign_signature:

Generating a signature 
==============================

eformsign_signature uses asymmetric key cryptography and elliptic curve cryptography.

.. tip:: 
   
   Elliptic curve cryptography is a public key encryption method and is currently the most popular method used in data encryption, digital authentication, etc.


The following describes how to generate a signature for each language: Java, Python, and PHP.

Java
-------

Convert the current time of the server to String (UTF-8) and sign using the private key issued to you in `Getting an API Key <#apikey>`__\. Then, convert the signed data into hex string.

.. note:: 

    The signature algorithm used is SHA256withECDSA.


Javascript(Node.JS)
------------------------

Ensure that Jsrsasign(https://kjur.github.io/jsrsasign/) npm is installed.

.. code:: Javascript

   npm install jsrsasign



Python
-------

Use the library for key generation in the link below. Install the library using the following command.

.. code:: python

   pip install https://github.com/warner/python-ecdsa/archive/master.zip


PHP
-------

Ensure that PHP OpenSSL library is installed and the keycheck.inc.php and test.php files in the following examples are located in the same path before proceeding.


Examples
---------------------

The following shows the examples for each language.


.. note:: 

   For execution_time, long type is used. Therefore, when entering the execution_time, add 'L' at the end of the excution time which were given with the issuance of the Access Token. 



.. code-tabs::

    .. code-tab:: java
        :title: Java

        import java.security.KeyFactory;
        import java.security.spec.PKCS8EncodedKeySpec;
        import java.security.PrivateKey;
        import java.security.Signature;
         
        //private key
        String privateKeyHexStr = "the private key(String) issued to you";
        KeyFactory keyFact = KeyFactory.getInstance("EC");
        PKCS8EncodedKeySpec psks8KeySpec = new PKCS8EncodedKeySpec(new BigInteger(privateKeyHexStr,16).toByteArray());
        PrivateKey privateKey = keyFact.generatePrivate(psks8KeySpec);
         
        //execution_time - the current server time
        long execution_time = new Date().getTime();
        String execution_time_str = String.valueOf(execution_time);
         
        //the generated eformsign_signature
        Signature ecdsa = Signature.getInstance("SHA256withECDSA");
        ecdsa.initSign(privateKey);
        ecdsa.update(execution_time_str.getBytes("UTF-8"));
        String eformsign_signature = new BigInteger(ecdsa.sign()).toString(16);
         
         
        //the current time and the signature value at the current time
        System.out.print("execution_time : "+execution_time);
        System.out.print("eformsign_signature : "+eformsign_signature);


    .. code-tab:: javascript
        :title: Javascript(Node.JS)

        const rs = require('jsrsasign');


        // User-Data-Here
        const execution_time  = Date.now()+"";
        const privateKeyHex = "the private key(String) issued to you";

        // User-Data-Here
        var privateKey = rs.KEYUTIL.getKeyFromPlainPrivatePKCS8Hex(privateKeyHex);

        // Sign
        var s_sig = new rs.Signature({alg: 'SHA256withECDSA'});
        s_sig.init(privateKey);
        s_sig.updateString(execution_time);
        var signature = s_sig.sign();
        console.log('data:', execution_time);
        console.log('eformsign_signature:', signature);


    .. code-tab:: python
        :title: Python

        import hashlib
        import binascii
         
        from time import time
        from ecdsa import SigningKey, VerifyingKey, BadSignatureError
        from ecdsa.util import sigencode_der, sigdecode_der
         
        # private key
        privateKeyHex = "the private key(String) issued to you"
        privateKey = SigningKey.from_der(binascii.unhexlify(privateKeyHex))
         
        # execution_time - current server time
        execution_time_int = int(time() * 1000)
        execution_time = str(execution_time_int)
          
        # the generated eformsign_signature
        eformsign_signature = privateKey.sign(execution_time.encode('utf-8'), hashfunc=hashlib.sha256, sigencode=sigencode_der)
          
        # the current time and the signature value at the current time
        print("execution_time : " + execution_time)
        print("eformsign_signature : " + binascii.hexlify(signature).decode('utf-8'))

    .. code-tab:: php
        :title: PHP - keycheck.inc.php

        <?php
        namespace eformsignECDSA;
  
        class PublicKey
        {
          
            function __construct($str)
            {
                $pem_data = base64_encode(hex2bin($str));
                $offset = 0;
                $pem = "-----BEGIN PUBLIC KEY-----\n";
                while ($offset < strlen($pem_data)) {
                    $pem = $pem . substr($pem_data, $offset, 64) . "\n";
                    $offset = $offset + 64;
                }
                $pem = $pem . "-----END PUBLIC KEY-----\n";
                $this->openSslPublicKey = openssl_get_publickey($pem);
            }
        }

        class PrivateKey
        {         

            function __construct($str)
            {
                $pem_data = base64_encode(hex2bin($str));
                $offset = 0;
                $pem = "-----BEGIN EC PRIVATE KEY-----\n";
                while ($offset < strlen($pem_data)) {
                    $pem = $pem . substr($pem_data, $offset, 64) . "\n";
                    $offset = $offset + 64;
                }
                $pem = $pem . "-----END EC PRIVATE KEY-----\n";
                $this->openSslPrivateKey = openssl_get_privatekey($pem);
            }
        }


        function getNowMillisecond()
        {
          list($microtime,$timestamp) = explode(' ',microtime());
          $time = $timestamp.substr($microtime, 2, 3);
          
          return $time;
        }
         
         
        function Sign($message, $privateKey)
        {
            openssl_sign($message, $signature, $privateKey->openSslPrivateKey, OPENSSL_ALGO_SHA256);
            return $signature;
        }
        ?>

    .. code-tab:: php
        :title: PHP - test.php

        <?php
        require_once __DIR__ . '/keycheck.inc.php';
 
        use eformsignECDSA\PrivateKey;
         
         
        define('PRIVATE_KEY', 'the private key(String)' issued to you);
         
         
        //setting a private key
        $privateKey = new PrivateKey(PRIVATE_KEY);
         
         
        //execution_time - current server time
        $execution_time = eformsignECDSA\getNowMillisecond();
         
         
        //the generated eformsign_signature
        $signature = eformsignECDSA\Sign(execution_time, $privateKey);
         
         
        //the current time and the signature value at the current time
        print 'execution_time : ' . execution_time . PHP_EOL;
        print 'eformsign_signature : ' . bin2hex($signature) . PHP_EOL;
        ?>



Open API list
=================

eformsign provides API for generating a signature and API for document creation and process.


API for generating a signature
------------------------------------

To generate a signature, use `Access Token API <https://app.swaggerhub.com/apis-docs/eformsign_api.en/eformsign_API_2.0/2.0#/token/post-api_auth-access_token>`_\. 

``POST``: `Issue Access Token <https://app.swaggerhub.com/apis-docs/eformsign_api.en/eformsign_API_2.0/2.0#/token/post-api_auth-access_token>`_\ 


Click
`here <https://app.swaggerhub.com/apis-docs/eformsign_api.en/eformsign_API_2.0/2.0#/token/post-api_auth-access_token>`__\  for more information about Access Token API.

.. caution:: 
   
   There is a time limit of 30 seconds when generating a signature. A signature must be generated and a token must be issued within 30 seconds. Also, the server time and the current time may not match. Check the 'execution_time' of response message received after calling Access Token API.

   .. code:: JSON

      { "code": "4000002", "ErrorMessage": "The validation time has expired.",     "execution_time": 1611538409405 }

   Enter 'execution_time' in the following `Example <https://app.swaggerhub.com/apis-docs/eformsign_api.en/eformsign_API_2.0/2.0#/token/post-api_auth-access_token>`__\.
   
   |image5| 

   Access Token can be issued for member. To get an Access Token for a member, enter 'member_id' together with the 'execution_time'. 
   
   |image6| 


   When the API is executed, Access Token will be issued and you will receive the following response.

   .. code:: JSON

      { "api_key": { "name": "Application_", "alias": "test", "company": { "company_id": "dec5418e58694d90a65d6c38e3d226db", "name": "Sample demo", "api_url": "https://kr-api.eformsign.com" } }, "oauth_token": { "expires_in": 3600, "token_type": "JWT", "refresh_token": "8fd0a3c1-44dc-4a03-96ad-01fa34cd159c", "access_token": "eyJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJlZm9ybXNpZ24uaWFtIiwiY29udGV4dCI6eyJjbGllbnRJZCI6IjY4MDk0ZWVhMjVhZjRhNjI5ZTI4ZGU5Y2ZlYzRlYmZjIiwiY2xpZW50S2V5IjoiZTNiM2IzZTUtMGEzMS00NTE1LWE5NzEtN2M4Y2FlNDI4NzZmIiwibWFuYWdlbWVudElkIjoiMzRhYWI4MDBjMmEwNDQwNThmZDRlZjc5OGFlY2RlY2EiLCJzY29wZXMiOiJzbWFydF9lZm9ybV9zY29wZSIsInR5cGUiOiJ1c2VyIiwidXNlck5hbWUiOiIzMmIzZDRmOC00MjdkLTRjZjQtOTZiYS1mYzAxNjIxNWRkNDciLCJ1c2VySWQiOiJhNTEyNGVkNmU2M2Y0OTMzOGJlOTA0MjVhNjFkYjlmNSIsInJlZnJlc2hUb2tlbiI6IjhmZDBhM2MxLTQ0ZGMtNGEwMy05NmFkLTAxZmEzNGNkMTU5YyJ9LCJjbGFpbSI6eyJjb21wYW55X2lkIjoiZGVjNTQxOGU1ODY5NGQ5MGE2NWQ2YzM4ZTNkMjI2ZGIiLCJhY2Nlc3Nfa2V5IjoiMzJiM2Q0ZjgtNDI3ZC00Y2Y0LTk2YmEtZmMwMTYyMTVkZDQ3In0sImV4cCI6MTYxMTU0MjIzNiwiaWF0IjoxNjExNTM4NjM2fQ.BltoXXBSabjXfpyLsZik9OZTE5XtLqe9lguMmJ_qfwZN1NyoVoxDqA5y1-_TLis7FvvNjfI1eegOroCZDZPFyXRaBxAj0CW8TijVjbhliJBuccHFyKXaJxmo_GMmTHYtxNNB1SUgLeFIrYROnpQndU8J7ZkfPDgYGwh1YSx-5s4" } }





.. caution:: 
   
   Register the issued API key by clicking the **Authorize** button(|image4|) in `here <https://app.swaggerhub.com/apis-docs/eformsign_api.en/eformsign_API_2.0/2.0#/token/post-api_auth-access_token>`__\. Note that you must enter **Base64-encoded** strings in the API key value. Go to https://www.base64encode.org/ and enter the issued API key to encode it.


.. note:: 
  
   Make sure to enter API key value by clicking the **Authorize** button of Access Token API.


---------------------------


API for document creation and process
------------------------------------------

After generating a signature, you can use the following document API to create new documents, to inquiry document information, or to download completed document files(PDF and audit trail certificate), or attached files.


.. caution:: 

   To use the document API, you need to get Access Token first. Enter the Access Token issued from `Access Token API <https://app.swaggerhub.com/apis-docs/eformsign_api.en/eformsign_API_2.0/2.0#/token/post-api_auth-access_token>`_\  by clicking the **Authorize** button(|image4|) in `here <https://app.swaggerhub.com/apis-docs/eformsign_api.en/eformsign_API_2.0/2.0#/>`__\. 


.. note:: 
  
   Make sure to enter Access Token value by clicking the **Authorize** button. 


The followings are `Document API <https://app.swaggerhub.com/apis-docs/eformsign_api.en/eformsign_API_2.0/2.0#/document>`_\  provided in eformsign.



``POST``: `Create a new document_member <https://app.swaggerhub.com/apis-docs/eformsign_api.en/eformsign_API_2.0/2.0#/document/post-api-documents>`_\ 

``POST``: `Create a new document_external recipients <https://app.swaggerhub.com/apis-docs/eformsign_api.en/eformsign_API_2.0/2.0#/document/post-api-documents-external>`_\ 

``GET``: `Inquiry document information <https://app.swaggerhub.com/apis-docs/eformsign_api.en/eformsign_API_2.0/2.0#/document/get-api-documents-DOCUMENT_ID>`_\

``GET``: `Download document files_PDF and Audit trail certificate <https://app.swaggerhub.com/apis-docs/eformsign_api.en/eformsign_API_2.0/2.0#/document/get-api-documents-DOCUMENT_ID-download_files>`_\

``GET``: `Download attached files <https://app.swaggerhub.com/apis-docs/eformsign_api.en/eformsign_API_2.0/2.0#/document/get-api-documents-DOCUMENT_ID-download_attach_files>`_\ 

``GET``: `Inquiry document list <https://app.swaggerhub.com/apis-docs/eformsign_api.en/eformsign_API_2.0/2.0#/document/get-api-documents>`_\ 

``POST``: `Inquiry document list <https://app.swaggerhub.com/apis-docs/eformsign_api.en/eformsign_API_2.0/2.0#/document/get-api-documents>`_\ 

``DELETE``: `Delete document <https://app.swaggerhub.com/apis-docs/eformsign_api.en/eformsign_API_2.0/2.0#/document/delete-api-documents>`_\ 

``POST``: `Resend a document_external recipients <https://app.swaggerhub.com/apis-docs/eformsign_api.en/eformsign_API_2.0/2.0#/document/post-api-documents-document_id-re_request_outsider>`_\ 

``GET``: `Inquiry available template lists <https://app.swaggerhub.com/apis-docs/eformsign_api.en/eformsign_API_2.0/2.0#/document/get-api-forms>`_\  

``DELETE``: `Delete template <https://app.swaggerhub.com/apis-docs/eformsign_api.en/eformsign_API_2.0/2.0#/document/get-api-forms>`_\  

``POST``: `Send in bulk_single template  <https://app.swaggerhub.com/apis-docs/eformsign_api.en/eformsign_API_2.0/2.0#/document/post-api-forms-mass_documents%3Ftemplate_id%3D-form_id>`_\  

``POST``: `Send in bulk_multiple templates <https://app.swaggerhub.com/apis-docs/eformsign_api.en/eformsign_API_2.0/2.0#/document/post-api-forms-mass_documents%3Ftemplate_id%3D-form_id>`_\  

``POST``: `Void document <https://app.swaggerhub.com/apis-docs/eformsign_api.en/eformsign_API_2.0/2.0#/document/post-api-forms-mass_documents%3Ftemplate_id%3D-form_id>`_\  

``GET``: `Usage status <https://app.swaggerhub.com/apis-docs/eformsign_api.en/eformsign_API_2.0/2.0#/document/post-api-forms-mass_documents%3Ftemplate_id%3D-form_id>`_\  

``POST``: `Reject document_members <https://app.swaggerhub.com/apis-docs/eformsign_api.en/eformsign_API_2.0/2.0#/document/post-api-forms-mass_documents%3Ftemplate_id%3D-form_id>`_\  

``POST``: `Reject document_non members <https://app.swaggerhub.com/apis-docs/eformsign_api.en/eformsign_API_2.0/2.0#/document/post-api-forms-mass_documents%3Ftemplate_id%3D-form_id>`_\  

-----------------


API for managing members and groups
--------------------------------------------

You can use the following API to manage members and groups.


.. caution:: 

   To use the document API, you need to get Access Token first. Enter the Access Token issued from `Access Token API <https://app.swaggerhub.com/apis-docs/eformsign_api.en/eformsign_API_2.0/2.0#/token/post-api_auth-access_token>`_\  by clicking the **Authorize** button(|image4|) in `here <https://app.swaggerhub.com/apis-docs/eformsign_api.en/eformsign_API_2.0/2.0#/>`__\. 

.. note:: 
  
   Make sure to enter Access Token value by clicking the **Authorize** button. 


The followings are `Member and group management API <https://app.swaggerhub.com/apis-docs/eformsign_api.en/eformsign_API_2.0/2.0#/members>`_\  provided in eformsign.


API for managing members
^^^^^^^^^^^^^^^^^^^^^^^^^^


``GET``: `Inquiry member list <https://app.swaggerhub.com/apis-docs/eformsign_api.en/eformsign_API_2.0/2.0#/members/get-api-members>`_\   

``PATCH``: `Edit member <https://app.swaggerhub.com/apis-docs/eformsign_api.en/eformsign_API_2.0/2.0#/members/patch-api-members-member_id>`_\  

``DELETE``: `Delete member <https://app.swaggerhub.com/apis-docs/eformsign_api.en/eformsign_API_2.0/2.0#/members/delete-api-members-member_id>`_\  


API for managing groups
^^^^^^^^^^^^^^^^^^^^^^^^


``GET``: `Inquiry group list <https://app.swaggerhub.com/apis-docs/eformsign_api.en/eformsign_API_2.0/2.0#/groups/get-api-groups>`_\  

``POST``: `Add group <https://app.swaggerhub.com/apis-docs/eformsign_api.en/eformsign_API_2.0/2.0#/groups/post-api-groups>`_\  

``PATCH``: `Edit group <https://app.swaggerhub.com/apis-docs/eformsign_api.en/eformsign_API_2.0/2.0#/groups/patch-api-groups>`_\  

``DELETE``: `Delete group <https://app.swaggerhub.com/apis-docs/eformsign_api.en/eformsign_API_2.0/2.0#/groups/delete-api-groups>`_\  



.. note:: 

    Click 
    `here <https://app.swaggerhub.com/apis-docs/eformsign_api.en/eformsign_API_2.0/2.0#/>`_\  for more information about each eformsign API.






API code
===================

API status code
----------------------

The API status code are as follows.

200
^^^^^^^^

===========  ===============  ===================================
Code         Description      Remark-
===========  ===============  ===================================
200          Success          Success
===========  ===============  ===================================


202
^^^^^^^^

===========  ====================  =========================================================================
Code         Description            Remark
===========  ====================  =========================================================================
2020001      Generating a PDF       -When downloading a PDF file, the file is generated asynchronously, 
                                     so it takes additional time to generate the PDF file after saving a document.
                                    -Downloadable when rerequesting within seconds to minutes.
===========  ====================  =========================================================================



400
^^^^^^^^

===========  =====================================  =================================================================================
Code         Description                             Remark
===========  =====================================  =================================================================================
4000001      When omitting a required input value    When the API's required input value (header or parameter value) is omitted                        
4000002      Authentication timeout                  When the API authentication request time has expired
4000003      No API key                              When the API key is deleted or incorrectly entered
4000004      No document                             When the document ID is incorrectly entered
4000005      No company                              When the company is deleted
===========  =====================================  =================================================================================


403
^^^^^^^^

===========  =======================================  ==========================================
Code         Description                               Remark
===========  =======================================  ==========================================
4030001      No permission to access                   When the API is disabled
4030002      Access token authentication error         When the access token is incorrect
4030003      Refresh token authentication error        When the refresh token is incorrect
4030004      Signature value authentication failure    When the signature value is incorrect
4030005      Unsupported API                           When calling an unsupported API
===========  =======================================  ==========================================


405
^^^^^^^^

===========  =====================  ======================================
Code         Description            Remark
===========  =====================  ======================================
4050001      Unsupported method     When calling an unsupported method
===========  =====================  ======================================


500
^^^^^^^^

===================  ===============  ===================================
Code                 Description      Remark
===================  ===============  ===================================
5000001~5000003      Server error     When a server error occurs
===================  ===============  ===================================

----------------------


User types
--------------

===================  ===============  ===============================================
Type                 Code             Description
===================  ===============  ===============================================
Member               01               Whether the user is a member
Non-member           02               Whether the user is a non-member
===================  ===============  ===============================================


Step types
--------------

=============  ===============  ===================================
Type           Code             Description
=============  ===============  ===================================
Start          00               Start step
Complete       01               Complete step
Approval       02               Approval step
External       03               External recipient step
Accept         04               Internal recipient step
Participant    05               Participant step
Reviewer       06               Reviewer step
Need to view   07               Need to view step
=============  ===============  ===================================


Document current status types
------------------------------

=========================  ===============  ==================================================
Type                       Code             Description
=========================  ===============  ==================================================
doc_tempsave               001              Draft (temporarily saved by the document creator)
doc_create                 002              Document created
doc_complete               003              Document completed
doc_update                 043              Document updated
doc_request_delete         047              Document requested to be deleted
doc_delete                 049              Document deleted
doc_request_revoke         040              Document requested to be voided
doc_revoke                 042              Document voided
doc_request_reject         045              Document requested to be declined
doc_request_participant    060              Document requested to a participant
doc_accept_participant     062              Document approved by a participant 
doc_reject_participant     061              Document declined by a participant
doc_request_reviewer       070              Document requested to a reviewer
doc_accept_reviewer        072              Document approved by a reviewer
doc_reject_reviewer        071              Document declined by a reviewer
=========================  ===============  ==================================================


Document next status types
------------------------------

===============  ===============  ==================================================
Type              Code             Description
===============  ===============  ==================================================
Draft             00               Document saved as draft in the start step
In progress       01               Document requested
Correcting        02               Document being corrected (member, document creator)
Completed         03               Document completed
Rejected          04               Document rejected by an approver/reviewer
Voided            05               Document voided
Void requested    06               Document requested to be voided
===============  ===============  ==================================================


Action types
--------------

=========================  ===============  =================================================================
Type                        Code             Description
=========================  ===============  =================================================================
doc_tempsave                 001              Saving a document as a draft
doc_create                   002              Creating a document
doc_complete                 003              Completing a document
doc_request_approval         010              Requesting a document for approval
doc_reject_approval          011              Declining a document approval request
doc_accept_approval          012              Approving a document approval request
doc_cancel                   013              Cancelling a document approval
doc_request_reception        020              Requesting a document to be approved by an internal recipient
doc_reject_reception         021              Requesting a document to be declined by an internal recipient
doc_accept_reception         022              An internal recipient approving a document approval request
doc_accept_tempsave          023              An internal recipient saving a document as a draft
doc_request_outsider         030              Requesting a document approval to an internal recipient
doc_reject_outsider          031              An external recipient declining a document
doc_accept_outsider          032              An external recipient approving a document
doc_rerequest_outsider       033              Rerequesting a document approval to an external recipient
doc_open_outsider            034              An external recipient opening a document
doc_outsider_tempsave        035              An external recipient saving a document as a draft
doc_request_revoke           040              Requesting a document to be voided
doc_refuse_revoke            041              Declining a document void request
doc_revoke                   042              Voiding a document
doc_update                   043              Correcting a document
doc_cancel_update            044              Cancelling a document correction
doc_request_reject           045              Requesting a document to be declined
doc_refuse_reject            046              Rejecting a document decline request
doc_request_delete           047              Requesting a document to be deleted
doc_refuse_delete            048              Rejecting a document deletion request
doc_delete                   049              Deleting a document
doc_complete_send_pdf        050              Sending a PDF file of a completed document
doc_transfer                 051              Transferring a document
doc_request_participant      060              Requesting a document to participant   
doc_reject_participant       061              Rejecting a document by participant  
doc_accept_participant       062              A participant approving a document  
doc_rerequest_participant    063              A participant(enon-member) rerequesting a document    
doc_open_participant         064              A participant(non-member) opening a document    
doc_request_reviewer         070              A reviewer requesting a document    
doc_reject_reviewer          071              A reviewer rejecting a document    
doc_request_reviewer         072              A reviewer approving a document    
doc_rerequest_reviewer       073              A reviewer(non-member) rerequesting a document    
doc_open_review              074              A reviewer(non-member) opening a document
=========================  ===============  =================================================================









.. |image1| image:: resources/companyinfo_companyid.png
   :width: 600px
.. |image2| image:: resources/column_icon.png
.. |image3| image:: resources/document_id.png
.. |image4| image:: resources/authorize_icon.png
.. |image5| image:: resources/execution_time.png
   :width: 650px
.. |image6| image:: resources/execution_time2.png
   :width: 450px

