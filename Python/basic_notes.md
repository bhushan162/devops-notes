# concept

- taking argument
    
    sys.argv[1]
    
- 

## interaction with Linux /bash

- subprocess.run()
    
    ```bash
    import subprocess
    
    # Running a native Linux command via Python to check disk space
    # capture_output=True   --> capture output as stdout 
    #  text=True            --> output into redable format 
    #  df --> disk free space 
    
    result = subprocess.run(["df", "-h"], capture_output=True, text=True)
    
    print("--- NATIVE LINUX OUTPUT ---")
    print(result.stdout)  # Prints the actual 'df -h' terminal output
    
    print("--- OS EXIT CODE ---")
    print(result.returncode)  # Will print 0 because the command succeeds
    ```
    
- os.environ
    
    it is used to set envrementy vairable 
    
    ```bash
    #setting vairable and its value 
    os.environ['NEW_VAR'] = 'value'
    
    # geeting vsairable value 
    os.environ['API_KEY']                                  #return keyerror if key is missing 
    
    #safer option
    os.environ.get('API_KEY', 'default_value')              #return NOne of default value 
    
    #deleting vairable 
    del os.environ['VAR_NAME']
    
    ```
    
- 

# scenario

- os.system() or subprocess() which is more secure and why?
    
    subprocess.run()
    
- chak disk space using automatio or if fails resturn msg write python script
    
    ```bash
    import subprocess
    
    # Running a native Linux command via Python to check disk space
    result = subprocess.run(["df", "-h"], capture_output=True, text=True)
    
    # If Linux exit code is NOT 0, if berween 1 to 255 means error relatd to that error code 
    if result.returncode != 0:                            
        print("🚨 CRITICAL ALARM: System command failed!")
        print(f"Kernel Error Message Logged:\n{result.stderr}")
    else:
    	print("--- NATIVE LINUX OUTPUT ---")
    	print(result.stdout)  # Prints the actual 'df -h' terminal output
    	
    	print("--- OS EXIT CODE ---")
    	print(result.returncode)  # Will print 0 because the command succeeds 
    ```