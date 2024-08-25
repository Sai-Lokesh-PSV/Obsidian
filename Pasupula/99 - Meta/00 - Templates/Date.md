---
creation date: <% tp.file.creation_date() %>
modification date: <% tp.file.last_modified_date("dddd Do MMMM YYYY HH:mm:ss") %> 
--- 
<< [[<% tp.date.now("YYYY-MM-DD", -1) %>]] |  [[<% tp.date.now("YYYY-MM-DD", 0) %>]] | [[<% tp.date.now("YYYY-MM-DD", 1) %>]] >> 
# <% tp.file.title %>

<% tp.date.now() %>
<% tp.date.now("YYYY-MM-DD", 7) %>
<% tp.date.now("YYYY-MM-DD", 7, "2021-04-09", "YYYY-MM-DD") %>
<% tp.date.now("dddd, MMMM Do YYYY", 0, tp.file.title, "YYYY-MM-DD") %>