## Configure to avoid Line Feed EOL character issues

### On Windows

This means that Git will process all text files and make sure that CRLF is replaced with LF when writing that file to the object database and turn all LF back into CRLF when writing out into the working directory. This is the recommended setting on Windows because it ensures that your repository can be used on other platforms while retaining CRLF in your working directory.

```powershell
git config --global core.autocrlf true
# Check it worked...
git config --global core.autocrlf
```


### On Linux
This means that Git will process all text files and make sure that CRLF is replaced with LF when writing that file to the object database. It will not, however, do the reverse. When you read files back out of the object database and write them into the working directory they will still have LFs to denote the end of line. This setting is generally used on Unix/Linux/OS X to prevent CRLFs from getting written into the repository. The idea being that if you pasted code from a web browser and accidentally got CRLFs into one of your files, Git would make sure they were replaced with LFs when you wrote to the object database.

```bash
git config --global core.autocrlf input
# Check it worked...
git config --global core.autocrlf
```

See “[Mind the end of your Line](https://adaptivepatchwork.com/2012/03/01/mind-the-end-of-your-line/)”.