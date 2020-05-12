export-things-todotxt
=====================

```
	set theFilePath to (path to desktop as Unicode text) & "Things export.txt"

	to replaceText(someText, oldItem, newItem)
		(*
	     replace all occurances of oldItem with newItem
	          parameters -     someText [text]: the text containing the item(s) to change
	                    oldItem [text, list of text]: the item to be replaced
	                    newItem [text]: the item to replace with
	          returns [text]:     the text with the item(s) replaced
	     *)
		set {tempTID, AppleScript's text item delimiters} to {AppleScript's text item delimiters, oldItem}
		try
			set {itemList, AppleScript's text item delimiters} to {text items of someText, newItem}
			set {someText, AppleScript's text item delimiters} to {itemList as text, tempTID}
		on error errorMessage number errorNumber -- oops
			set AppleScript's text item delimiters to tempTID
			error errorMessage number errorNumber -- pass it on
		end try
		
		return someText
	end replaceText

	on extractThings()
		set theList to {"Inbox", "Today", "Next", "Scheduled"}
		
		local extractedThings, theListItem, toDo, toDos, tdName, tdDueDate, tdNotes, noteParagraph, prToDo, prToDos, prtdName, prtdDueDate, prtdNotes, prnoteParagraph
		
		set extractedThings to ""
		
		tell application "Things"
			repeat with theListItem in theList
				set toDos to to dos of list theListItem
				
				repeat with toDo in toDos
					
					set tdName to the name of toDo
					
					if (due date of toDo) is not missing value then
						set due_date to (due date of toDo)
						set m to (month of due_date)
						set m to text -2 through -1 of ("0" & (m as integer as text))
						
						set d to (day of due_date)
						set d to text -2 through -1 of ("0" & (d as integer as text))
						set tdDueDate to " Due:" & (year of due_date) & "-" & m & "-" & d
					else
						set tdDueDate to ""
					end if
					
					if (project of toDo) is not missing value then
						set tdProject to " +" & (name of project of toDo)
					else
						set tdProject to ""
					end if
					
					if (area of toDo) is not missing value then
						set todoArea to " @" & (name of area of toDo)
					else
						set todoArea to ""
					end if
					
					if (tag names of toDo) is not equal to "" then
						set tags_of_todo to tag names of toDo
						set todo_tags to " #" & replaceText(tags_of_todo, ", ", " #") of me
					else
						set todo_tags to ""
					end if
					
					set extractedThings to tdName & tdProject & todoArea & todo_tags & tdDueDate & linefeed & extractedThings
					
				end repeat
				
			end repeat
		end tell
		
		return extractedThings
		
	end extractThings

	set scriptOutput to ""

	if application "Things" is running then
		
		set scriptOutput to scriptOutput & extractThings()
		
		set theFile to (open for access file (theFilePath) with write permission)
		set eof of theFile to 0
		write scriptOutput to theFile
		close access theFile
		
	else
		
		try
			set theFile to (open for access file (theFilePath))
			set scriptOutput to (read theFile)
			close access theFile
			
		on error
			
			tell application "Finder" to delete file theFilePath
			set scriptOutput to ""
			
		end try
	end if
```
