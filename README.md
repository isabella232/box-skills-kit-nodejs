# Box Skills Kit Node.js SDK
Official SDK for writing Box Skills in Node.js

You can visit the [Box Skills Developer Documentation](https://developer.box.com/docs/box-skills) for more information about Box Skills.

The Box Skills Kit SDK allows your application to receive and interpret event notifications from Box, retreive data from Box, and write to metadata in Box. 

The Box Skills Kit Node.js SDK contains two classes:
* The **Files Reader** class interprets incoming events from Box and can retrieve files from Box for processing
* The **Skills Writer** class writes data to a set of metadata templates in Box

Using these two classes, your application can retrieve files as they're uploaded to Box for processing and write the outputs of the processing to Box's metadata. This metadata information can be used to drive other Box functionality, such as search and preview.

## FilesReader
A helpful class to capture file information from the incoming event notification from Box and to retrieve the file's content from Box

**FilesReader( boxEvent.body ) CONSTRUCTOR**

| Functions                                           | Returns          | Description                                                                                                                                                     |
|-----------------------------------------------------|------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------|
| getFileContext ()                       | FileContext JSON | Returns a JSON containing fileId, fileName, fileFormat, fileType, fileSize, fileDownloadURL, fileReadToken, fileWriteToken, skillId, requestId for use in code. |
| validateFormat (allowedFileFormatsList) | Boolean          | Helper function to check if a given file is eligible to be processed by the skill as per the list of allowed formats.                                           |
| validateSize ( allowedMegabytesNum)     | Boolean          | Helper function to check if a given file is eligible to be processed by the skill as per the size limit.                                                        |
| getContentBase64 ( )                    | String           | Returns a Blob of the file content in Base64 encoding. Note: ML providers generally have a size limit on passing raw file content as a string                   |
| getContentStream ( )                    | Stream           | Returns a Read Stream to be passed to read file directly from box. Note: Some ML providers support passing file read streams.                                   |
| getBasicFormatFileURL ( )               | String           | Same as filesReader.getFileContext().fileDownloadURL but in BasicFormat (See Note below)                                                                        |
| getBasicFormatContentBase64 ( )         | String           | Same as filesReader.getFileContext().getContentBase64() but in BasicFormat (See Note below)                                                                     |
| getBasicFormatContentStream ( )         | Stream           | Same as filesReader.getFileContext().getContentStream() but in BasicFormat(See Note below)                                                                      |

Note: BasicFormat functions allows you to access files stored in Box in another format, which may be more accepted by ML providers. The provided basic formats are Audio files→.mp3, Document/Image files→.jpeg, Video files→.mp4. Caution should be excercised using BasicFormats for certain large files as it involves a time delay, and your skill code or skills-engine request may time out before the converted format is fetched.
 
 ## SkillsWriter
 A helpful class to write metadata to Box to display in cards for Topics, Transcripts, Timelines, Errors and Statuses for any file for which a Box Skill processes.

**SkillsWriter( fileContext )  (see FilesReader.getFileContext above)**

| Functions                                                                                           | Returns       | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
|-----------------------------------------------------------------------------------------------------|---------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| createTopicsCard ( topicsDataList, optionalFileDuration, optionalCardTitle )           | DataCard JSON | This card is used show multiple keywords that are relevant to the topic of the file. topicsDataList is of form: [{ text: 'text1'}, { text: 'text2'} ]. In case of an audio/video file you can also optionally show a timeline of where that words in the file, by passing down topicsDataList in the form: [{ text: 'text1', appears: [{ start: 0.0, end: 1.0 }]}, { text: 'text2', appears: [{ start: 1.0, end: 2.0 }]} ] and by providing the optionalFileDuration of the entire file. The default title of the card is 'Topics' unless optionalCardTitle is provided.                                                                                                           |
| createTranscriptsCard ( transcriptsDataList, optionalFileDuration, optionalCardTitle ) | DataCard JSON | This card is used to show sentences such as speaker transcripts in audio/video or OCR in images/documents. transcriptDataList is of form: [{ text: 'sentence1'}, { text: 'sentence2'} ]. In case of an audio/video file you can also optionally show a timeline of where that sentence is spoken in the file, by passing down transcriptsDataList in the form: [{ text: 'sentence1', appears: [{ start: 0.0, end: 1.0 }]}, { text: 'sentence2', appears: [{ start: 1.0, end: 2.0 }]} ] The default title of the card is 'Transcript' unless optionalCardTitle is provided.                                                                                                         |
| createFacesCard ( facesDataList, optionalFileDuration, optionalCardTitle )             | DataCard JSON | This card is used to show thumbnails of recognized faces or objects in the file, along with associated texts or sentences. facesDataList is of form: [{ text: 'text1', 'image_url' : thumbnailUri1}, { text: 'text2', 'image_url' : thumbnailUri2} ]. In case of an audio/video file you can also optionally show a timeline of where that face appears in the file, by passing down transcriptsDataList in the form: [{ text: 'text1', appears: [{ start: 0.0, end: 1.0 }]}, { text: 'text2', appears: [{ start: 1.0, end: 2.0 }]} ] and by providing the optionalFileDuration of the entire file. The default title of the card is 'Faces' unless optionalCardTitle is provided. |
| saveProcessingCard ( optionalCallback )                                             | Null          | Shows UI card with message: "We're preparing to process your file. Please hold on!". This is used for temporarily letting your users know that your skill is under progress. You can pass an optionalCallback function to print or log success in your code once the card has been saved.                                                                                                                                                                                                                                                                                                                                                                                          |
| saveErrorCard ( error, optionalMessage, optionalCallback )                 | Null          | Show UI card with error message as noted in SkillsErrorEnum table. Used to notify user if any kind of failure occurs while running your skills code. Shows card as per the default message with each code. An optionalMessage will overwrite the the default message of the Erro enum. You can pass an optionalCallback function to print or log success in your code once the card has been saved.                                                                                                                                                                                                                                                                                                       |
| saveDataCards ( listofDataCardJSONs, optionalCallback)                                 | Null          | Shows all the cards passed in listofDataCardJSONswhich can be of formatted as Topics,Transcripts or Faces. Will override any existing pending or error status cards in the UI for that file version.                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
 ## Error Enum
 Possible error messages to display in an ErrorStatus card.
 
| Functions                                      | Description                                                                                             |
|------------------------------------------------|---------------------------------------------------------------------------------------------------------|
| SkillsErrorEnum.FILE_PROCESSING_ERROR   | Shows card: "We're sorry, something went wrong with processing the file."                               |
| SkillsErrorEnum.INVALID_FILE_SIZE       | Shows card: "Something went wrong with processing the file. This file size is currently not supported." |
| SkillsErrorEnum.INVALID_FILE_FORMAT     | Shows card: "Something went wrong with processing the file. Invalid information received."              |
| SkillsErrorEnum.INVALID_EVENT           | Shows card: "Something went wrong with processing the file. Invalid information received."              |
| SkillsErrorEnum.NO_INFO_FOUND           | Shows card: "We're sorry, no skills information was found."                                             |
| SkillsErrorEnum.INVOCATIONS_ERROR       | Shows card: "Something went wrong with running this skill or fetching its data."                        |
| SkillsErrorEnum.EXTERNAL_AUTH_ERROR | Shows card: "Something went wrong with running this skill or fetching its data."                        |
| SkillsErrorEnum.BILLING_ERROR       | Shows card: "Something went wrong with running this skill or fetching its data."                        |
| SkillsErrorEnum.UNKNOWN             | Shows card: "Something went wrong with running this skill or fetching its data."                        |
