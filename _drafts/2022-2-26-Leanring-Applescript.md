---
layout: post
title: Learning Applescript
date: 2022-3-8 15:43:44
category: personal
tags: 
---

Applescript version of my **Post to Jekyll** workflow.

```
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
		-- set dateString to "date: " & (current date)
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