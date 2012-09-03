This readme file explains the changes made for the challenge 1751.

Overview

The current version of the databasedotcom ruby gem contains a bug that prevents it from 
updating records if you are using a Database.com org (it works fine with a Force.com org). 
We want you to fix this bug and submit a pull request to our repo.
We've put together a small databasedotcom rails app that you can use for testing as well as a short video to 
help you get a Database.com org up and running quickly. The error occurs on line 44 of the accounts_controller 
when updating the record and throws Net::HTTPBadRequest. 
This is the stack trace of the error as well as some additional information buried at the end of this thread.


Bug Research and Changes:
As I was trying to figure out what was causing this error I came across this link:
 http://www.salesforce.com/us/developer/docs/api_rest/index_Left.htm#StartTopic=Content/dome_sobject_create.htm?SearchType=Stem


Basically, the approach is that instead of using the PATCH request directly, the request could be 
performed using POST requests. Well, I decided to try it out and that ended up fixing the problem.
The idea is to append the http parameter '_HttpMethod=PATCH' to the URL of the POST request.

What I did then was to add an attribute to the Databasedotcom::Client class (patch_using_post).
This attribute would allow the developer using this code to switch between performing real PATCH request
and performing PATCH requests using the POST method.

I'm not completely sure that this is the way to fix this issue, but at least it solves it and it could
help in other situation like those where proxy servers don't support PATCH requests.


The only change to for the sample rails test application https://github.com/cloudspokes/databasedotcom-rails-test
would be patch_using_post: true to the databasedotcom.yml file. 
Of course, installation of this new version of the GEM needs to be done first.


Here is a quick list of the changes:
====================================

    lib/databasedotcom/client.rb:
         This file has been changed to add the patch_using_post attribute and use it for biz logic in the 
         Databasedotcom::Client.http_patch method.

	spec/fixtures/databasedotcom.yml:
	     Added the patch_using_post attribute for testing purposes.
	
	spec/lib/client_spec.rb:
	     Added/Modified the corresponding tests for coverage of the addition of the patch_using_post attribute.
	     Note: The file diff.txt details the changes made.
	
	     Modified tests:
	         Databasedotcom::Client 
	           configuration from environment variables
	               takes configuration information from the environment, if present
	               takes configuration information from a URL
	           from a yaml file
	               takes configuration information from the specified file
	           from a hash
	               takes configuration information from the hash
	               accepts symbols in the hash
	         
		     Databasedotcom::Client precedence
				   prefers the environment configuration to the YAML configuration

	     Added tests:
	        Databasedotcom::Client defaults
	           defaults to using real patch requests
	
	        Databasedotcom::Client authentiation with a materialized class
	          #update
	              with proper fields performing patch request using post
	              with attributes from a hash
	              persists the updated changes with names as symbols
	              applies type coercions before serializing
	              applies type coercions with Dates represented as Strings
	
	          #upsert
	              with a valid external field performing patch request using post
	                 with a non-existent external id
	                    creates a new record with the external id and the specified attributes
	                 with an existing external id
	                     updates attributes in the existing object
	                 with multiple choice   
	                     raises an Databasedotcom::SalesForceError
	
	              with an invalid external field performing patch request using post
	                 raises an Databasedotcom::SalesForceError
	              
	          #http_patch performing patch request using post   
	              upserts the data to the specified path
	              puts parameters into the path
	              includes the headers in the request
	              raises SalesForceError
	              
