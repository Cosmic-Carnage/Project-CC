---
layout: default
title: Spacebook
permalink: /spacebook
---
<html>
<head>
    <title>Cosmic Carnage</title>
    <style>
        canvas {
        background-color: black;
        display: block;
        margin: 0 auto;
    }
        body {
            font-family: Arial, sans-serif;
            background-color: #f2f2f2;
            margin: 0;
            padding: 0;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            height: 100vh;
        }
        h1 {
            text-align: center;
        }
        #createPlayerForm {
            background-color: gray;
            border-radius: 5px;
            padding: 20px;
            box-shadow: 0px 0px 10px rgba(0, 0, 0, 0.1);
        }
        label {
            font-weight: bold;
        }
        input[type="text"],
        input[type="number"] {
            width: 100%;
            padding: 10px;
            margin-bottom: 10px;
            border: 1px solid #ccc;
            border-radius: 4px;
            color: black;
        }
        input[type="submit"] {
            background-color: gray;
            color: black;
            padding: 10px 20px;
            border: none;
            border-radius: 4px;
            cursor: pointer;
        }
    </style>
</head>
<html>
<head>
    <title>Spacebook</title>
</head>
<body>
    <form id="uploadForm" enctype="multipart/form-data" method="post">
        <input type="file" name="image" accept=".jpg, .png, .gif">
        <input type="text" name="fileName" placeholder="Enter file name">
        <input type="submit" value="Upload Image">
    </form>
    <script>
        document.getElementById("uploadForm").addEventListener("submit", function (event) {
            event.preventDefault();
            const form = new FormData(this);
            // Create a new XMLHttpRequest object
            const xhr = new XMLHttpRequest();
            // Define the HTTP method and the URL
            xhr.open("POST", "https://cosmic-backend.stu.nighthawkcodingsociety.com/image", true);
            // Set CORS headers
            xhr.setRequestHeader("Access-Control-Allow-Origin", "*");
            // Handle the response
            xhr.onload = function () {
                if (xhr.status === 200) {
                    console.log("Image request successful!");
                } else {
                    console.error("Image request failed with status: " + xhr.status);
                }
            };
            // Send the request
            xhr.send(form);
        });
    </script>
</body>
