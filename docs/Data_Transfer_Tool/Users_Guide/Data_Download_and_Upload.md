#Data Downloads and Uploads from the command line.

## Downloads

### Downloading Data Using a Manifest File

A convenient way to download multiple files from the GDC is to use a manifest file generated by the GDC Data Portal. After generating a manifest file (see [Preparing for Data Download and Upload](Preparing_for_Data_Download_and_Upload.md) for instructions), initiate the download using the GDC Data Transfer Tool by supplying the **-m** or **--manifest** option, followed by the location and name of the manifest file. OS X users can drag and drop the manifest file into Terminal to provide its location.

The following is an example of a command for downloading files from GDC using a manifest file:

	gdc-client download -m  /Users/JohnDoe/Downloads/gdc_manifest_6746fe840d924cf623b4634b5ec6c630bd4c06b5.txt

### Downloading Data Using GDC File UUIDs

The GDC Data Transfer Tool also supports downloading of one or more individual files using UUID(s) instead of a manifest file. To do this, enter the UUID(s) after the download command:

	gdc-client download 22a29915-6712-4f7a-8dba-985ae9a1f005

Multiple UUIDs can be specified, separated by a space:

	gdc-client download e5976406-473a-4fbd-8c97-e95187cdc1bd fb3e261b-92ac-4027-b4d9-eb971a92a4c3

### Resuming a Failed Download

The GDC Data Transfer Tool supports resumption of interrupted downloads. To resume an incomplete download, repeat the download of the manifest or UUID(s) in the same folder as the initial download.  Failed downloads will appear in the destination folder with a .partial extension. This feature allows users the ability to identify quickly where the download stopped.  For large downloads this feature can let the user identify where the download was interrupted  and edit the manifest accordingly.

	gdc-client download f80ec672-d00f-42d5-b5ae-c7e06bc39da1

### Downloading Controlled-Access Data

A user authentication token is required for downloading Controlled-Access Data from GDC. Tokens can be obtained from the GDC Data Portal (see instructions in [Obtaining an Authentication Token](Preparing_for_Data_Download_and_Upload.md#obtaining-an-authentication-token)). Once downloaded, the token *file* can be passed to the GDC Data Transfer Tool using the **-t** or **--token-file** option:

	gdc-client download -m gdc_manifest_e24fac38d3b19f67facb74d3efa746e08b0c82c2.txt -t gdc-user-token.2015-06-17T09-10-02-04-00.txt


### Directory structure of downloaded files

The directory in which the files are downloaded will include folders named by the file UUID. Inside these folders, along with the the data and zipped metadata or index files, will exist a logs folder. The logs folder contains state files that insure that downloads are accurate and allow for resumption of failed or prematurely stopped downloads.  While a download is in progress a file will have a .partial extension.  This will also remain if a download failed.  Once a file is finished downloading the extension will be removed.  If an identical manifest is retried another attempt will be made to download files containing a .partial extension.  

	C501.TCGA-BI-A0VR-10A-01D-A10S-08.5_gdc_realn.bam.partial  logs 	 

## Uploads

### Uploading Data Using a Manifest File

GDC Data Transfer Tool supports uploading molecular data using a manifest file to the Data Submission Portal. The manifest file for submittable data files can be retrieved from the GDC Data Submission Portal, or directly from the GDC Submission API given a submittable data file UUID. The user authentication token file needs to be specified using the **-t** or **--token-file** option.

First, generate an upload manifest, either using the GDC Data Submission Portal, or [using a call](/API/Users_Guide/Submission.md#upload-manifest) to the GDC Submission API `manifest` endpoint (as in the following example):

```Manifest
export token=ALPHANUMERICTOKEN-01234567890+AlPhAnUmErIcToKeN=0123456789-ALPHANUMERICTOKEN-01234567890+AlPhAnUmErIcToKeN=0123456789-ALPHANUMERICTOKEN-01234567890+AlPhAnUmErIcToKeN=0123456789-ALPHANUMERICTOKEN-01234567890+AlPhAnUmErIcToKeN=0123456789-ALPHANUMERICTOKEN-01234567890+AlPhAnUmErIcToKeN=0123456789-ALPHANUMERICTOKEN-01234567890+AlPhAnUmErIcToKeN=0123456789-ALPHANUMERICTOKEN-01234567890+AlPhAnUmErIcToKeN=0123456789-ALPHANUMERICTOKEN-01234567890+AlPhAnUmErIcToKeN=0123456789-ALPHANUMERICTOKEN-01234567890+AlPhAnUmErIcToKeN=0123456789-ALPHANUMERICTOKEN-01234567890+AlPhAnUmErIcToKeN=0123456789-ALPHANUMERICTOKEN-01234567890+AlPhAnUmErIcToKeN=0123456789-ALPHANUMERICTOKEN-01234567890+AlPhAnUmErIcToKeN=0123456789-ALPHANUMERICTOKEN-01234567890+AlPhAnUmErIcToKeN=0123456789-ALPHANUMERICTOKEN-01234567890+AlPhAnUmErIcToKeN=0123456789-ALPHANUMERICTOKEN-01234567890+AlPhAnUmErIcToKeN=0123456789-ALPHANUMERICTOKEN-01234567890+AlPhAnUmErIcToKeN=0123456789-ALPHANUMERICTOKEN-01234567890+AlPhAnUmErIcToKeN=0123456789-ALPHANUMERICTOKEN-01234567890+AlPhAnUmErIcToKeN=0123456789-ALPHANUMERICTO

curl --header "X-Auth-Token: $token" 'https://api.gdc.cancer.gov/submission/CGCI/BLGSP/manifest?ids=460ad2fe-5a7f-4797-9e18-336d33e21444' >manifest.yml
```
```Upload
gdc-client upload --manifest manifest.yml --token-file token.txt
```

### Uploading Data Using a GDC File UUID

The GDC Data Transfer Tool also supports uploading molecular data using a file UUID. The tool will first make a request to get the filename and project id from GDC API, and then upload the corresponding file from the current directory.

	gdc-client upload cd939bdd-b607-4dd4-87a6-fad12893932d -t token.txt

### Resuming a Failed Upload

By default, GDC Data Transfer Tool uses multipart transfer to upload files. If an upload failed but some parts were transmitted successfully, a resume file will be saved with the filename *resume\_[manifest\_filename]*. Running the upload command again will resume the transfer of only those parts of the file that failed to upload in the previous attempt.

	gdc-client upload -m manifest.yml -t token

### Deleting Previously Uploaded Data

Previously uploaded data can be replaced with new data by deleting it first using the **--delete** switch:

	gdc-client upload -m manifest.yml -t token --delete


## Recurrent Transfers of Very Large Datasets over High-speed Networks

Institutions that regularly transfer very large volumes of data between GDC facilities (located in Chicago, IL, USA) and a geographically remote location over gigabit+ networks may benefit from using the UDT mode of the GDC Data Transfer Tool. **UDT mode** is an advanced feature that uses [UDT](http://udt.sourceforge.net/), or User Datagram Protocol (UDP)-based Data Transfer, instead of the ubiquitous [Transmission Control Protocol (TCP) protocol](https://tools.ietf.org/html/rfc793). Please <a href="https://gdc.cancer.gov/support#gdc-help-desk">contact the GDC Helpdesk</a> if you are interested in learning more about this feature.


## Troubleshooting

### Invalid Token

An error message about an &#39;invalid token&#39; means that a new authentication token needs to be obtained from the GDC Data Portal or the GDC Data Submission Portal as described in [Preparing for Data Download and Upload](Preparing_for_Data_Download_and_Upload.md).


	 403 Client Error: FORBIDDEN: {
		  "message": "Your token is invalid or expired, please get a new token from GDC Data Portal"
		  }

### dbGaP Permissions Error

Users may see the following error message when attempting to download a file from GDC:

	 403 Client Error: FORBIDDEN: {
		  "message": "You don't have access to the data: Please specify a X-Auth-Token"
		  }


This error message indicates that the user does not have dbGaP access to the project to which the file belongs. Instructions for requesting access from dbGaP can be found [here](https://gdc.cancer.gov/access-data/obtaining-access-controlled-data/).

### File Availability Error

Users may also see the following error message when attempting to download a file from GDC:

	 403 Client Error: FORBIDDEN: {
		"message": "You don't have access to the data: Requested file abd28349-92cd-48a3-863a-007a218de80f does not allow read access"
	  	}

This error message means that the file is not available for download. This may be because the file has not been uploaded or released yet or that it is not a file entity.

### GDC Upload Privileges Error

Users may see the following error message when attempting to upload a file:

	 Can't upload: {
		 "message": "You don't have access to the data: You don't have create role to do 'upload'"
	     }

This means that the user has dbGaP read access to the data, but does not have GDC upload privileges. Users can contact [The database of Genotypes and Phenotypes (dbGaP) ](https://www.ncbi.nlm.nih.gov/gap) to request upload privileges.

### File in Uploaded State Error

Re-uploading a file may return the following error:

	 Can't upload: {
		  "message": "File in uploaded state, upload not allowed"
		  }

To resolve this issue, delete the file using the **--delete** switch before re-uploading.

### Microsoft Windows Executable Error

Attempting to run gdc-client.exe by double-clicking it in the Windows Explorer will produce a window that blinks once and disappears.

This is normal, the executable must be run using the command prompt. Click 'Start', followed by 'Run' and type 'cmd' into the text bar.  Then navigate to the path containing the executable using the 'cd' command.     
