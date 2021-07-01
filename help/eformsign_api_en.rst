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
------------------------------------------------------

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


3. Enter the alias and application name in the **Create an API key** pop-up and then click the **Save** button.

.. image:: resources/apikey3.PNG
    :width: 700
    :alt: Create an API key pop-up


4. Click the **View key** in the list to view the API key and Private key.

.. image:: resources/apikey4.PNG
    :width: 700
    :alt: View key button

.. image:: resources/apikey5.PNG
    :width: 700
    :alt: API key and Private key



.. note:: **Editing an API key**

    You can change the alias and application name by clicking the **Edit** button in the API key list. You can also disable/enable the key by clicking the **Status** toggle button.

.. note:: **Deleting an API key**

    You can delete an API key by clicking the **Delete** button in the API key list.


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
        execution_time = int(time() * 1000)
          
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

To generate a signature, use `Access Token API <https://app.swaggerhub.com/apis/eformsign_api/eformsign_API_2.0/2.0_auth>`_\. 

``POST``: `Issue Access Token <https://app.swaggerhub.com/apis/eformsign_api/eformsign_API_2.0/2.0_auth#/eformsign/post-api_auth-access_token>`_\ 


Click
`here <https://app.swaggerhub.com/apis/eformsign_api/eformsign_API_2.0/2.0_auth>`__\  for more information about Access Token API.

.. caution:: 
   
   There is a time limit of 30 seconds when generating a signature. A signature must be generated and a token must be issued within 30 seconds. Also, the server time and the current time may not match. Check the 'execution_time' of response message received after calling Access Token API.

   .. code:: JSON

      { "code": "4000002", "ErrorMessage": "The validation time has expired.",     "execution_time": 1611538409405 }

   Enter 'execution_time' in the following `Example <https://app.swaggerhub.com/apis/eformsign_api/eformsign_API_2.0/2.0_auth>`__\.
   
   |image5| 

   Access Token can be issued for member. To get an Access Token for a member, enter 'member_id' together with the 'execution_time'. 
   
   |image6| 


   When the API is executed, Access Token will be issued and you will receive the following response.

   .. code:: JSON

      { "api_key": { "name": "Application_", "alias": "test", "company": { "company_id": "dec5418e58694d90a65d6c38e3d226db", "name": "Sample demo", "api_url": "https://kr-api.eformsign.com" } }, "oauth_token": { "expires_in": 3600, "token_type": "JWT", "refresh_token": "8fd0a3c1-44dc-4a03-96ad-01fa34cd159c", "access_token": "eyJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJlZm9ybXNpZ24uaWFtIiwiY29udGV4dCI6eyJjbGllbnRJZCI6IjY4MDk0ZWVhMjVhZjRhNjI5ZTI4ZGU5Y2ZlYzRlYmZjIiwiY2xpZW50S2V5IjoiZTNiM2IzZTUtMGEzMS00NTE1LWE5NzEtN2M4Y2FlNDI4NzZmIiwibWFuYWdlbWVudElkIjoiMzRhYWI4MDBjMmEwNDQwNThmZDRlZjc5OGFlY2RlY2EiLCJzY29wZXMiOiJzbWFydF9lZm9ybV9zY29wZSIsInR5cGUiOiJ1c2VyIiwidXNlck5hbWUiOiIzMmIzZDRmOC00MjdkLTRjZjQtOTZiYS1mYzAxNjIxNWRkNDciLCJ1c2VySWQiOiJhNTEyNGVkNmU2M2Y0OTMzOGJlOTA0MjVhNjFkYjlmNSIsInJlZnJlc2hUb2tlbiI6IjhmZDBhM2MxLTQ0ZGMtNGEwMy05NmFkLTAxZmEzNGNkMTU5YyJ9LCJjbGFpbSI6eyJjb21wYW55X2lkIjoiZGVjNTQxOGU1ODY5NGQ5MGE2NWQ2YzM4ZTNkMjI2ZGIiLCJhY2Nlc3Nfa2V5IjoiMzJiM2Q0ZjgtNDI3ZC00Y2Y0LTk2YmEtZmMwMTYyMTVkZDQ3In0sImV4cCI6MTYxMTU0MjIzNiwiaWF0IjoxNjExNTM4NjM2fQ.BltoXXBSabjXfpyLsZik9OZTE5XtLqe9lguMmJ_qfwZN1NyoVoxDqA5y1-_TLis7FvvNjfI1eegOroCZDZPFyXRaBxAj0CW8TijVjbhliJBuccHFyKXaJxmo_GMmTHYtxNNB1SUgLeFIrYROnpQndU8J7ZkfPDgYGwh1YSx-5s4" } }





.. caution:: 
   
   Register the issued API key by clicking the **Authorize** button(|image4|) in `here <https://app.swaggerhub.com/apis/eformsign_api/eformsign_API_2.0/2.0_auth>`__\. Note that you must enter **Base64-encoded** strings in the API key value. Go to https://www.base64encode.org/ and enter the issued API key to encode it.


.. note:: 
  
   Make sure to enter API key value by clicking the **Authorize** button of Access Token API.


---------------------------


API for document creation and process
------------------------------------------

After generating a signature, you can use the following document API to create new documents, to inquiry document information, or to download completed document files(PDF and audit trail certificate), or attached files.


.. caution:: 

   To use the document API, you need to get Access Token first. Enter the Access Token issued from `Access Token API <https://app.swaggerhub.com/apis/eformsign_api/eformsign_API_2.0/2.0_auth>`_\  by clicking the **Authorize** button(|image4|) in `here <https://app.swaggerhub.com/apis/eformsign_api/eformsign_API_2.0/2.0_general>`__\. 


.. note:: 
  
   Make sure to enter Access Token value by clicking the **Authorize** button. 


The followings are `Document API <https://app.swaggerhub.com/apis/eformsign_api/eformsign_API_2.0/2.0_general>`_\  provided in eformsign.



``POST``: `Create a new document_member <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/default/post-api-documents>`_\ 

``POST``: `Create a new document_external recipients <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/eformsign/post-api-documents-external>`_\ 

``GET``: `Inquiry document information <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/eformsign/get-api-documents-DOCUMENT_ID>`_\

``GET``: `Download document files_PDF and Audit trail certificate <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/eformsign/get-api-documents-DOCUMENT_ID-download_files>`_\

``GET``: `Download attached files <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/eformsign/get-api-documents-DOCUMENT_ID-download_attach_files>`_\ 

``GET``: `Inquiry document list <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/default/get-api-documents>`_\ 

``DELETE``: `Delete document <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/default/delete-api-documents>`_\ 

``POST``: `Resend a document_external recipients <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/default/post-api-documents-document_id-re_request_outsider>`_\ 

``GET``: `Inquiry available template lists <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/default/get-api-forms>`_\  

``POST``: `Send in bulk <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/default/post-api-forms-mass_documents%3Ftemplate_id%3D-form_id>`_\  

-----------------


API for managing members and groups
--------------------------------------------

You can use the following API to manage members and groups.


.. caution:: 

   To use the document API, you need to get Access Token first. Enter the Access Token issued from `Access Token API <https://app.swaggerhub.com/apis/eformsign_api/eformsign_API_2.0/2.0_auth>`_\  by clicking the **Authorize** button(|image4|) in `here <https://app.swaggerhub.com/apis/eformsign_api/eformsign_API_2.0/2.0_general>`__\. 


.. note:: 
  
   Make sure to enter Access Token value by clicking the **Authorize** button. 


The followings are `Member and group management API <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/members>`_\ provided in eformsign.


API for managing members
^^^^^^^^^^^^^^^^^^^^^^^^^^


``GET``: `Inquiry member list <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/default/get-api-members>`_\   

``PATCH``: `Edit member <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/default/patch-api-members-member_id>`_\  

``DELETE``: `Delete member <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/default/delete-api-members-member_id>`_\  


API for managing groups
^^^^^^^^^^^^^^^^^^^^^^^^


``GET``: `Inquiry group list <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/default/get-api-groups>`_\  

``POST``: `Add group <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/default/post-api-groups>`_\  

``PATCH``: `Edit group <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/default/patch-api-groups>`_\  

``DELETE``: `Delete group <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/default/delete-api-groups>`_\  



.. note:: 

    Click 
    `here <https://app.swaggerhub.com/apis/eformsign_api/eformsign_API_2.0/2.0_general>`__\  for more information about each eformsign document API.






API code
===================

API status code
----------------------

The API status code are as follows.

200
^^^^^^^^

===========  ===============  ===================================
Code         Description      Remark
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


Step types
--------------

===========  ===============  ===================================
Type         Code             Description
===========  ===============  ===================================
Start         00               Start step
Complete      01               Complete step
Approval      02               Approval step
External      03               External recipient step
Accept        04               Internal recipient step
Participant   05               Participant
Reviewer      06               Reviewer
===========  ===============  ===================================


User types
--------------

===================  ===============  ===============================================
Type                 Code             Description
===================  ===============  ===============================================
Internal member      01               Whether the user is an internal member or not
External recipient   02               Whether the user is an external user or not
===================  ===============  ===============================================

Status types
--------------

===============  ===============  ==================================================
Type              Code             Description
===============  ===============  ==================================================
Draft             00               Document saved as draft at start step
In pregress       01               Document requested for approval, requested to member or external recipient
Correcting        02               Document being corrected (member, document creator)
Completed         03               Document completed
Rejected          04               Document rejected by approver, member, or external recipient
Voided            05               Document voided
Void requested    06               Document requested to void to member
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
doc_rerequest_participant    063              A participant(external recipient) rerequesting a document    
doc_open_participant         064              A participant(external recipient) opening a document    
doc_request_reviewer         070              A reviewer requesting a document    
doc_reject_reviewer          071              A reviewer rejecting a document    
doc_request_reviewer         072              A reviewer approving a document    
doc_rerequest_reviewer       073              A reviewer(external recipient) rerequesting a document    
doc_open_review              074              A reviewer(external recipient) opening a document
=========================  ===============  =================================================================





Detailed status type
--------------------------------

=========================  ===============  ======================================================================
Type                        Code             Description
=========================  ===============  ======================================================================
doc_tempsave                001              Draft (when a document is saved as a draft by the initial creator)
doc_create                  002              Document created
doc_complete                003              Document completed
doc_update                  043              Document modified
doc_request_delete          047              Document deletion requested
doc_delete                  049              Document deleted
doc_request_revoke          040              Document void requested
doc_revoke                  042              Document voided
doc_request_reject          045              Document decline requested
doc_request_approval        010              Document approval requested
doc_accept_approval         012              Document approval request approved
doc_reject_approval         011              Document approval request declined
doc_cancel                  013              Document approval request canceled
doc_request_reception       020              Document approval requested to internal recipient
doc_accept_reception        022              Document approval request approved by internal recipient
doc_reject_reception        021              Document approval request declined by internal recipient
doc_request_outsider        030              Document approval requested to external recipient
doc_accept_outsider         032              Document approved request approved by external recipient
doc_reject_outsider         031              Document approval request declined by external recipient
doc_request_participant     060              Document requested to participant
doc_accept_participant      062              Document accepted by participant
doc_reject_participant      061              Document rejected by participant
doc_request_reviewer        070              Document requested to reviewer    
doc_accept_reviewer         072              Document accepted by reviewer 
doc_reject_reviewer         071              Document rejected by reviewer
=========================  ===============  ======================================================================








.. |image1| image:: resources/companyinfo_companyid.png
   :width: 600px
.. |image2| image:: resources/column_icon.png
.. |image3| image:: resources/document_id.png
.. |image4| image:: resources/authorize_icon.png
.. |image5| image:: resources/execution_time.png
   :width: 450px
.. |image6| image:: resources/execution_time2.png
   :width: 450px

