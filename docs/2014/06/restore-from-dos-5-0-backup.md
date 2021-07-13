# Restore from DOS 5.0 backup
It never ceases to amaze me what some customers require from us. Recently a customer called wanting to restore files from an old backup they had. As it turns out this backup was stored on a floppy disk in a safety deposit box for who knows how long.

The customer was able to get he files off the floppy before I even took a look. The backup consisted of 2 files, BACKUP.001 and CONTROL.001. It turns out this is a backup from DOS 5.0, this requires the use of restore.com to restore the files.

Lucky for us you can download a copy of a compatible restore utility that goes by the same name (restore.com) from (http://mindprod.com/products4.html) unlucky for us it won’t run on any recent version of Windows however it will run under DosBox (http://www.dosbox.com/)

If you create a directory on your hard drive and extract the contents of restore64.zip (From mindprod) you can mount it in DosBox using mount c: c:\restore\ (Assuming you are mounting the directory c:\restore to the c drive)

You will also need to create a floppy image containing BACKUP.001 and CONTROL.001. It didn’t like a path being specified. Simply create a new image using WinImage (http://www.winimage.com/)

Add BACKUP.001 and CONTROL.001 to the image and save it as a vfd file. You can then mount it in DosBox using the command imgmount a: <vfd location> -t floppy

Then it is simply a matter of running 
```
c:\restore.com a: c:\*.* /s
```
if everything goes ok you should see output like below:
![](/img/dosrestore.png)

Of course then I had the fun task of converting everything from Works 4.0 to a doc. I managed to do this using a virtual machine, and Microsoft Works 95. Trying to use Office and the works converter didn’t work, it either failed with a nondescript error message or just displayed garbage.