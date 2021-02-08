----------------------------
Using eformsign Webhook
----------------------------

This is a feature that sends event information to the customer's system/service when an event occurs in eformsign. By configuring Webhook, event information is sent to the customer's webhook endpoint in the HTTP POST method.

.. tip:: 

    A Webhook endpoint refers to the customer's client callback URL. It allows you to get eformsign event information without making unnecessary calls, which differs from the Open API method where calls are continuously made to check for changes (polling).


Getting started
====================


.. _webhook:

Getting a Webhook key
-------------------------------

1. Log in to eformsign as the company administrator and then go to **[Integration] > [API / Webhook]** in the sidebar menu.

.. image:: resources/apikey1.PNG
    :width: 700
    :alt: Integration > API/Webhook menu


2. Select the **[Manage Webhook]** tab and then click the **New Webhook** button.

.. image:: resources/webhook2.PNG
    :width: 700
    :alt: New Webhook button


3. In the New Webhook pop-up, enter the Name, URL, and the List of affected templates, and make sure that the status is in enabled status. Then, click the **Register** button.

.. image:: resources/webhook3.PNG
    :width: 700
    :alt: New Webhook pop-up

4. Click the **View key** button in the Webhook list to view the Webhook public key.

.. image:: resources/webhook4.PNG
    :width: 700
    :alt: Webhook View key button

.. image:: resources/webhook5.PNG
    :width: 700
    :alt: View key



.. caution:: 

   If you click the **Regenerate key** button, the Webhook public key will be issued to you again and you will not be able to use the previous key.

.. note:: **Editing Webhook information**

     You can change the information of a Webhook by clicking the **Edit** button in the Webhook list.


.. note:: **Deleting a Webhook**

    You can delete a Webhook by clicking the **Delete** button in the Webhook list.   



5. Click the **Test** button in the Webhook list to send a test Webhook and get the returned results.

.. image:: resources/webhook6.PNG
    :width: 700
    :alt: Webhook Test 

The following is a json file used for testing purposes.

.. code:: json

	{
	"webhook_id" : "Webhook ID",
	"webhook_name" : "Webhook name",
	"company_id" : "company ID",
	"event_type" : “document”,
	"document" : {
	  "id" : “test_doc_id”,
	   "template_id" : “test_template_id”,
	   "template_version" : “1”,
	   "document_history_id" : “test_document_history_id”,
	   "doc_status" : “doc_create”,
	   "editor_id" : "user ID",
	   "updated_date" : "current time(UTC Long)"
	}
	}
	Test URL : Webhook URL




Generating a signature
==========================


The following describes how to generate a signature for each language: Java, Python, and PHP.

Java
-------

Check the event information received from the eformsign server by using the public key issued to you in `Getting a Webhook Key <#webhook>`__\ to verify whether the event was called normally from eformsign.


.. note:: 

  The signature algorithm used is SHA256withECDSA.


Python
-------

Use the library for key generation in the link below. Install the library using the following command.

.. code:: python

   pip install https://github.com/warner/python-ecdsa/archive/master.zip


PHP
-------

Make sure that the keycheck.inc.php and test.php files in the following examples are located in the same path before proceeding.


Examples
---------------------


The following shows the examples for each language.

.. code-tabs::

    .. code-tab:: java
        :title: Java

        import java.io.*;
		import java.math.BigInteger;
		import java.security.*;
		import java.security.spec.X509EncodedKeySpec;
		 
		....
		/**
		 *  Reads the header and body in the request.
		 *
		 */
		 
		 
		//1. get eformsign signature
		//eformsignSignature is contained in the request header.
		String eformsignSignature = request.getHeader("eformsign_signature");
		 
		 
		//2. get request body data
		// Converts the data in the body to string to verify the eformsign signature.
		String eformsignEventBody = null;
		StringBuilder stringBuilder = new StringBuilder();
		BufferedReader bufferedReader = null;
		 
		try {
		    InputStream inputStream = request.getInputStream();
		    if (inputStream != null) {
		        bufferedReader = new BufferedReader(new InputStreamReader(inputStream));
		        char[] charBuffer = new char[128];
		        int bytesRead = -1;
		        while ((bytesRead = bufferedReader.read(charBuffer)) > 0) {
		            stringBuilder.append(charBuffer, 0, bytesRead);
		        }
		    }
		 } catch (IOException ex) {
		    throw ex;
		 } finally {
		    if (bufferedReader != null) {
		        try {
		            bufferedReader.close();
		        } catch (IOException ex) {
		            throw ex;
		        }
		    }
		 }
		eformsignEventBody = stringBuilder.toString();
		 
		 
		 
		 
		//3. publicKey configuration
		String publicKeyHex = "the issued Public Key(String)";
		KeyFactory publicKeyFact = KeyFactory.getInstance("EC");
		X509EncodedKeySpec x509KeySpec = new X509EncodedKeySpec(new BigInteger(publicKeyHex,16).toByteArray());
		PublicKey publicKey = publicKeyFact.generatePublic(x509KeySpec);
		 
		//4. verify
		Signature signature = Signature.getInstance("SHA256withECDSA");
		signature.initVerify(publicKey);
		signature.update(eformsignEventBody.getBytes("UTF-8"));
		if(signature.verify(new BigInteger(eformsignSignature,16).toByteArray())){
		    //verify success
		    System.out.println("verify success");
		    /*
		     *Events are handled here.
		     */
		}else{
		    //verify fail
		    System.out.println("verify fail");
		}


    .. code-tab:: python
        :title: Python

        import hashlib
		import binascii
		 
		from ecdsa import VerifyingKey, BadSignatureError
		from ecdsa.util import sigencode_der, sigdecode_der
		from flask import request
		 
		 
		...
		# Reads the header and boy in the request.
		# 1. get eformsign signature
		# eformsignSignature is contained in the request header.
		eformsignSignature = request.headers['eformsign_signature']
		 
		 
		# 2. get request body data
		# Converts the data in the body to string to verify the eformsign signature.
		data = request.json
		 
		 
		# 3. publicKey configuration
		publicKeyHex = "the issued public key"
		publickey = VerifyingKey.from_der(binascii.unhexlify(publicKeyHex))
		 
		 
		# 4. verify
		try:
		    if publickey.verify(eformsignSignature, data.encode('utf-8'), hashfunc=hashlib.sha256, sigdecode=sigdecode_der):
		        print("verify success")
		        # Events are handled here.
		except BadSignatureError:
		    print("verify fail")

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
         
        function Verify($message, $signature, $publicKey)
        {
            return openssl_verify($message, $signature, $publicKey->openSslPublicKey, OPENSSL_ALGO_SHA256);
        }
        ?>

    .. code-tab:: php
        :title: PHP - test.php

        <?php
        require_once __DIR__ . '/keycheck.inc.php';
        use eformsignECDSA\PublicKey;
         
        define('PUBLIC_KEY', 'input the issued public key.');
        ...
        /*
         *  Reads the header and body in the request.
         *
         */
         
         
        //1. get eformsign signature
        //eformsignSignature is contained in the request header.
        $eformsignSignature = $_SERVER['HTTP_eformsign_signature'];
         
         
        //2. get request body data
        // Reads the data in the body to verify the eformsign signature.
        $eformsignEventBody = json_decode(file_get_contents('php://input'), true);
         
         
        //3. publicKey configuration
        $publicKey = new PublicKey(PUBLIC_KEY);
         
         
        //4. verify
        $ret = - 1;
        $ret = eformsignECDSA\Verify(MESSAGE, $eformsignSignature, $publicKey);
          
        if ($ret == 1) {
            print 'verify success' . PHP_EOL;
            /*
             * Events are handled here.
             */
        } else {
            print 'verify fail' . PHP_EOL;
        }
         ...
          
        ?>


Webhook list
====================

By configuring the following Webhook, you can receive the information in the webhook endpoint when the configured event occurs in eformsign. 

The following is `Webhook <https://app.swaggerhub.com/apis/eformsign_api/eformsign_API_2.0/Webhook>`_\ provided in eformsign.


``POST``: `/webhook document event <https://app.swaggerhub.com/apis/eformsign_api/eformsign_API_2.0/Webhook#/default/post-webhook-document-event>`_\  Send document events

``POST``: `/webhook pdf <https://app.swaggerhub.com/apis/eformsign_api/eformsign_API_2.0/Webhook#/default/post-webhook-pdf>`_\  Send PDF generation events


Click 
`here <https://app.swaggerhub.com/apis/eformsign_api/eformsign_API_2.0/Webhook>`__\ for more information about each eformsign Webhook.



Webhook list
=================

eformsign provides **document** events and **PDF generation** events as Webhook events.


Document events
------------------

This event occurs when a document is created or the status of a document is changed in eformsign.


.. table:: 

   ================ ====== ================
   Name             Type   Description
   ================ ====== ================
   id               String Document ID
   template_id      String Template ID
   template_name    String Template name
   template_version String Template title
   workflow_seq     int    Workflow order
   workflow_name    String Workflow name
   history_id       String Document history ID
   status           String Document status
   editor_id        String Creator ID
   updated_date     long   Document update time
   ================ ====== ================


Refer to the following for the meaning of each document status in event data.

.. _status: 

.. table:: 

   ========================== ==================
   Name                       Description
   ========================== ==================
   doc_create                 Document created
   doc_tempsave               Document saved as a draft
   doc_request_approval       Document approval requested
   doc_accept_approval        Document approval request approved
   doc_reject_approval        Document approval request declined
   doc_request_external       Document sent to external recipient
   doc_remind_external        Document resent to external recipient
   doc_open_external          Document opened by external recipient
   doc_accept_external        Document reviewed by external recipient
   doc_reject_external        Document declined by external recipient
   doc_request_internal       Document sent to internal recipient
   doc_accept_internal        Document reviewed by internal recipient
   doc_reject_internal        Document declined by internal recipient
   doc_tempsave_internal      Document saved as a draft by internal recipient
   doc_cancel_request         Document approval request cancelled
   doc_reject_request         Document requested to be declined
   doc_decline_cancel_request Document decline request rejected
   doc_delete_request         Document requested to be deleted 
   doc_decline_delete_request Document deletion request rejected
   doc_deleted                Document deleted
   doc_complete               Document completed
   ========================== ==================


PDF generation events
------------------------------

The following is a list of events generated when the PDF file of a document is created in eformsign.

.. table:: 

   ===================== ====== ================
   Name                  Type   Description
   ===================== ====== ================
   document_id           String Document ID
   template_id           String Template ID
   template_name         String Template name
   template_version      String Template title
   workflow_seq          int    Workflow order
   workflow_name         String Workflow name
   document_history_id   String Document history ID
   document_status       String Document status
   ===================== ====== ================


For the meaning of each document status in event data, refer to `the document status section <#status>`__\.