### Summary
We use the PBE - Password-based encryption algorithm to protect our config file. Its characteristic is that the password is controlled by the user itself, without any physical media; the random number (here we call salt) hash multiple encryption to ensure the security of the data. According to the definition of PBE, the key to decrypt is to obtain the password used for encryption.

We want to be able to get password at runtime. So maybe we can follow the pattern below：

If the service is deployed to VM, get password and salt from stdin

If the service is deployed to CaaS, get password and salt from environment variable

### Motivation


### Guide-level explanation
Because @dz-1 has implemented the config decryption integration in the config module. We can implement different types of decoder by configuring config.yml. This is similar to the functionality of the service module, but we can't use the service module. Because if the configuration module depends on the service module, there will be a dead loop of maven dependency.

We can define the following decoders to use in different runtime environments：

1. ManualAESDecryptor: can be used when user want to using stdin to get password. Since the config module will be invoke during server initialization process, the password can be got when server startup by stdin.
2. AutoAESDecryptor: can be used when user want to get password and salt from environment variable automatically. Devopt people need to provide the environment variable. When server startup, invoke System.getenv() to get password

Those two can be the implementation of Decryptor.

In conclusion, three are three working modes. User can choose which one to use throw setting the config.yml. The config.yml can be configured as the following for different scenario

1. No decryption demand, do not provide decryptorClass in the config.yml

        exclusionConfigFileList:
          - openapi
          - swagger
          - values
          - status
       
2. Need to deploy to VM

        exclusionConfigFileList:
          - openapi
          - swagger
          - values
          - status
        decryptorClass:
          com.networknt.decrypt.ManualAESDecryptor
        
    In this way, user need to input the password in console when server start.

    i.e.

    Password for config decryption:
    
    If the password is invalid, throw exception

3. Need to deploy to CaaS

        exclusionConfigFileList:
          - openapi
          - swagger
          - values
          - status
        decryptorClass:
          com.networknt.decrypt.AutoAESDecryptor
     
    In this way, the environment variables need to be provided.
    
    If the password is invalid, throw exception

### Reference-level explanation


### Drawbacks


### Rationale and Alternatives


### Unresolved questions
Since the decryption process has already integrated into config module and the config module will be invoke when server startup. Should we throw the exception to prevent server start if fail to decrypt configuration files?
