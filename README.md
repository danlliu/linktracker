# linktracker

## About

linktracker is a web app that keeps track of meeting links so you don't have to. Just enter your meeting title, link
, password (if it has one), and date/time, and it will keep track of that. You can filter your links by upcoming
 (within 24 hours), today (all events for today regardless of time), and starred.
 
linktracker was developed with HTML, Javascript, Bootstrap, and Vue.js.

## Version History
**v1.1.2 (March 24, 2021):** 
updated to Bootstrap 5.0! The following changes come as a result of that:
- Updated forms to have floating labels. 
- Updated dropdown for filtering to match the dark mode theme when dark mode is enabled. 
- Changed the appearance of toast notifications to make them less intrusive.

Other changes:
- Moved version number (and link to GitHub) to the left side to keep it with the header content.
- Changed the star from a button to an anchor tag, thus avoiding the "focused" state when it is clicked.
---  
**v1.1.1 (March 24, 2021):** 
quality of life fixes. Made links open in a new tab. 
Added transitions when switching between light and dark mode.
 ---  
**v1.1.0 (March 23, 2021):** 
added dark mode! you can enable dark mode using the button in the lower right of the screen. 
this affects all components. 
 ---  
**v1.0.2 (March 23, 2021):** 
fixed a bug where "Today" would show all repeating events that started before the current day, regardless of it they repeated on today's weekday. 
--- 
**v1.0.1 (March 22, 2021):** 
added notifications on the upper right to improve UI discoverability
---  
**v1.0.0 (March 22, 2021):** released the first version!
