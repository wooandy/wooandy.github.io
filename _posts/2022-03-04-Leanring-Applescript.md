---
layout: post
title: Learning Applescript - Automating my "Post to Jekyll" Workflow
date: 2022-03-04 18:00:01
category: personal
tags: 
---

Below is a Applescript to automate my **Post to Jekyll** workflow in BBEdit. This is essentially a combination of 2 separate scripts - Insert Front Matter and Rename Active Document[^1].

Firstly, it asks for the Post title.
![Post Title Dialog](https://s3.amazonaws.com//wookieweblog-files/post-title-dialog.jpg)

And then, the script will automagically insert the Front Matter to the beginning of the document. This is the main reason why I created this script.
![Added Front Matter](https://s3.amazonaws.com//wookieweblog-files/added-front-matter.jpg)

And finally it will use the post title as the document name, or you can change it, if you want to. 
![Rename active document](https://s3.amazonaws.com//wookieweblog-files/rename-active-document-dialog.jpg)


## How does the script work
* When the script runs, the explicit [**run handler**](https://developer.apple.com/library/archive/documentation/AppleScript/Conceptual/AppleScriptLangGuide/conceptual/ASLR_about_handlers.html#//apple_ref/doc/uid/TP40000983-CH206-SW15) will be executed first, display a dialog asking the user to input the Post title.
* **getTimeInHoursAndMinutes()** to get today's date and time as a string from ([current date](https://developer.apple.com/library/archive/documentation/AppleScript/Conceptual/AppleScriptLangGuide/reference/ASLR_cmds.html#//apple_ref/doc/uid/TP40000983-CH216-SW39))
* Assemble the Front Matter block with the entered Post title and current date and time.
* **findAndReplaceInText()** to replace space " " with dash "-" in the Post title. We don't want space in our document's name.
* Rename the document to date+post-title.      


<br>

##### Complete Applescript listing, you can download (File: [Insert Front Matter and Rename Document.scpt](https://gist.github.com/wooandy/b62599d01919b53112eb373ee40b898e)) from my Github
```applescript
on getTimeInHoursAndMinutes()
	
	-- Get the "hour"
	set timeStr to time string of (current date)
	
	set Pos to offset of ":" in timeStr
	set theHour to characters 1 thru (Pos - 1) of timeStr as string
	set timeStr to characters (Pos + 1) through end of timeStr as string
	
	-- Get the "minute"
	set Pos to offset of ":" in timeStr
	set theMin to characters 1 thru (Pos - 1) of timeStr as string
	set timeStr to characters (Pos + 1) through end of timeStr as string
	
	-- Get the "second"
	set Pos to offset of " " in timeStr
	set theSec to characters 1 thru (Pos - 1) of timeStr as string
	set timeStr to characters (Pos + 1) through end of timeStr as string
	
	set am_pm to timeStr
	
	if (am_pm is "PM") then
		set theHour to (theHour + 12) as string
	end if
	
	return (theHour & ":" & theMin & ":" & theSec) as string
	
end getTimeInHoursAndMinutes

on findAndReplaceInText(theText, theSearchString, theReplacementString)
	set AppleScript's text item delimiters to theSearchString
	set theTextItems to every text item of theText
	set AppleScript's text item delimiters to theReplacementString
	set theText to theTextItems as string
	set AppleScript's text item delimiters to ""
	return theText
end findAndReplaceInText


on run
	tell application "BBEdit"
		activate
		set the_text to text of front document
		set dashLine to "---"
		set layoutString to "layout: post"
		display dialog "Post title?" default answer "untitled"
		set titleEntered to the text returned of result
		set titleString to "title: " & titleEntered
		set {year:y, month:m, day:d, time:t} to (current date)
		set timeString to my getTimeInHoursAndMinutes()
		set dateString to "date: " & y & "-" & (m as number) & "-" & d & " " & timeString
		set categoryString to "category: personal"
		set tagString to "tags: "
		
		set the text of front document to ¬
			dashLine & return ¬
			& layoutString & return ¬
			& titleString & return ¬
			& dateString & return ¬
			& categoryString & return ¬
			& tagString & return ¬
			& dashLine & return ¬
			& the_text & return ¬
			as text
		normalize line endings of text of front document
		
		set titleEntered to my findAndReplaceInText(titleEntered, " ", "-")
		set fileString to y & "-" & (m as number) & "-" & d & "-" & titleEntered as string
		set old_name to name of text window 1
		set dialog_result to display dialog ¬
			"Rename active document:" default answer (fileString) ¬
			buttons {"Cancel", "Rename"} default button 2 ¬
			with icon note
		if button returned of dialog_result = "Rename" then
			set new_name to text returned of dialog_result
			set d to active document of text window 1
			if (d's on disk) then
				set the_file to d's file
				tell application "Finder"
					set name of the_file to new_name
					set name extension of the_file to "md"
				end tell
			else -- it's a document that has never been saved
				set name of d to new_name
			end if
		end if
		
	end tell
	
end run
```

[^1]: [Rename Active Document](https://daringfireball.net/2004/10/rename_active_document) Applescript is directly adopted from [Daring Fireball](https://daringfireball.net/).
