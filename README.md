# Database-Programming-Project
Database Programming Project Note taking management server 
Back end code 
-- Table: Attachment
CREATE TABLE IF NOT EXISTS Attachment (
AttachementID INT NOT NULL AUTO_INCREMENT,
AttachementType VARCHAR(45),
PRIMARY KEY (AttachementID)
);
-- Table: Device
CREATE TABLE IF NOT EXISTS Device (
DeviceID INT NOT NULL AUTO_INCREMENT,
DeviceName VARCHAR(45),
Paltform_PaltformID INT NOT NULL,
PRIMARY KEY (DeviceID),
FOREIGN KEY (Paltform_PaltformID) REFERENCES
Paltform (PaltformID)
);

-- Table: Folder
CREATE TABLE IF NOT EXISTS Folder (
FolderID INT NOT NULL AUTO_INCREMENT,
FolderName VARCHAR(45) NOT NULL,
DateCreated DATE NOT NULL,
DateModified DATE NOT NULL,

User_UserID INT NOT NULL,
PRIMARY KEY (FolderID),
FOREIGN KEY (User_UserID) REFERENCES User (UserID)
);

-- Table: NoteAttachment
CREATE TABLE IF NOT EXISTS NoteAttachment (
NoteAttachmentID INT NOT NULL AUTO_INCREMENT,
Notes_NoteID INT NOT NULL,
Notes_Folder_FolderID INT NOT NULL,
Attachment_AttachementID INT NOT NULL,
PRIMARY KEY (NoteAttachmentID),
FOREIGN KEY (Notes_NoteID, Notes_Folder_FolderID)
REFERENCES Notes (NoteID, Folder_FolderID),
FOREIGN KEY (Attachment_AttachementID) REFERENCES
Attachment (AttachementID)
);

-- Table: NoteFormatting
CREATE TABLE IF NOT EXISTS NoteFormatting (
formatingID INT NOT NULL AUTO_INCREMENT,
StyleName VARCHAR(45),
PRIMARY KEY (formatingID)
);
-- Table: NoteTags

CREATE TABLE IF NOT EXISTS NoteTags (
NoteTagID INT NOT NULL AUTO_INCREMENT,
Tags_TagID INT NOT NULL,
Notes_NoteID INT NOT NULL,
Notes_Folder_FolderID INT NOT NULL,
PRIMARY KEY (NoteTagID),
FOREIGN KEY (Tags_TagID) REFERENCES Tags (TagID),
FOREIGN KEY (Notes_NoteID, Notes_Folder_FolderID)
REFERENCES Notes (NoteID, Folder_FolderID)
);

-- Table: Notes
CREATE TABLE IF NOT EXISTS Notes (
NoteID INT NOT NULL AUTO_INCREMENT,
Title VARCHAR(45) NOT NULL,
Content VARCHAR(100),
CreationDate DATE,
LastModifiedDate DATE NOT NULL,
isPublic TINYINT NOT NULL,
Folder_FolderID INT NOT NULL,
NoteFormatting_formatingID INT NOT NULL,
PRIMARY KEY (NoteID),
FOREIGN KEY (Folder_FolderID) REFERENCES Folder
(FolderID),
FOREIGN KEY (NoteFormatting_formatingID) REFERENCES
NoteFormatting (formatingID)

);

-- Table: Paltform
CREATE TABLE IF NOT EXISTS Paltform (
PaltformID INT NOT NULL AUTO_INCREMENT,
PaltformName VARCHAR(45),
PlatformType VARCHAR(45),
PRIMARY KEY (PaltformID)
);

-- Table: SearchHistory
CREATE TABLE IF NOT EXISTS SearchHistory (
SearchID INT NOT NULL AUTO_INCREMENT,
SearchQuery VARCHAR(100),
SearchTimestamp DATETIME,
User_UserID INT NOT NULL,
PRIMARY KEY (SearchID),
FOREIGN KEY (User_UserID) REFERENCES User (UserID)
);

-- Table: Tags
CREATE TABLE IF NOT EXISTS Tags (
TagID INT NOT NULL AUTO_INCREMENT,
TagName VARCHAR(45),
PRIMARY KEY (TagID)

);

-- Table: User
CREATE TABLE IF NOT EXISTS User (
UserID INT NOT NULL AUTO_INCREMENT,
Username VARCHAR(45) NOT NULL,
Email VARCHAR(45) NOT NULL,
Password VARCHAR(45) NOT NULL,
PRIMARY KEY (UserID)
);
Middle-tier
Procedures

CREATE DEFINER=`root`@`localhost` PROCEDURE
`createNewNote`(IN p_username VARCHAR(45), IN p_selectedFolder
VARCHAR(45), IN p_newNoteTitle VARCHAR(45))
BEGIN
DECLARE v_folderId INT;

SELECT FolderID INTO v_folderId FROM Folder WHERE
FolderName = p_selectedFolder;

IF v_folderId IS NOT NULL THEN
INSERT INTO Notes (Title, Content, CreationDate,
LastModifiedDate, isPublic, Folder_FolderID,
NoteFormatting_formatingID) VALUES (p_newNoteTitle, &#39;&#39;, CURDATE(),
CURDATE(), 1, v_folderId, 1);
END IF;
END

CREATE DEFINER=`root`@`localhost` PROCEDURE `insertFolder`(IN
p_username VARCHAR(45), IN p_folderName VARCHAR(45))
BEGIN
DECLARE v_userId INT;

SELECT UserID INTO v_userId FROM User WHERE Username =
p_username;

IF v_userId IS NOT NULL THEN

INSERT INTO Folder (FolderName, DateCreacted, DateModified,
User_UserID) VALUES (p_folderName, CURDATE(), CURDATE(),
v_userId);
END IF;
END

CREATE DEFINER=`root`@`localhost` PROCEDURE `InsertUser`(IN
p_username VARCHAR(45), IN p_email VARCHAR(45), IN p_password
VARCHAR(45))
BEGIN
INSERT INTO User (Username, Email, Password) VALUES
(p_username, p_email, p_password);
END

CREATE DEFINER=`root`@`localhost` PROCEDURE `isValidLogin`(IN
p_username VARCHAR(45), IN p_password VARCHAR(45))
BEGIN
SELECT EXISTS(SELECT 1 FROM User WHERE Username =
p_username AND Password = p_password) AS isValid;
END

CREATE DEFINER=`root`@`localhost` PROCEDURE
`LoadNoteContent`(IN p_username VARCHAR(255), IN p_folderName
VARCHAR(255), IN p_noteTitle VARCHAR(255))
BEGIN
SELECT n.Content
FROM Notes n
JOIN Folder f ON n.Folder_FolderID = f.FolderID
JOIN User u ON f.User_UserID = u.UserID
WHERE f.FolderName = p_folderName

AND n.Title = p_noteTitle
AND u.Username = p_username;
END

CREATE DEFINER=`root`@`localhost` PROCEDURE
`populateFolderList`(IN p_username VARCHAR(45))
BEGIN
SELECT CONCAT(FolderName, &#39; (Created: &#39;, DateCreacted, &#39;,
Modified: &#39;, DateModified, &#39;)&#39;) AS folderInfo
FROM Folder f
JOIN User u ON f.User_UserID = u.UserID
WHERE u.Username = p_username;
END

CREATE DEFINER=`root`@`localhost` PROCEDURE
`SaveNoteContent`(IN p_username VARCHAR(255), IN p_folderName
VARCHAR(255), IN p_noteTitle VARCHAR(255), IN p_content TEXT)
BEGIN
UPDATE Notes n
JOIN Folder f ON n.Folder_FolderID = f.FolderID
JOIN User u ON f.User_UserID = u.UserID
SET n.Content = p_content
WHERE f.FolderName = p_folderName
AND n.Title = p_noteTitle
AND u.Username = p_username;
END

Triggers

CREATE TRIGGER after_note_insert
AFTER INSERT ON Notes
FOR EACH ROW
BEGIN
UPDATE Folder
SET DateModified = CURDATE()
WHERE FolderID = NEW.Folder_FolderID;
END;
