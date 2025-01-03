# JustWatch's watchlist to letterboxd
IMPORTANT: You need to be on https://www.justwatch.com/us/lists/my-lists and display the movies in list format as you can see on this photo

![image](https://github.com/user-attachments/assets/5cbc86e6-56d2-4bb8-b4d8-c8b863e5234b)

Then, open the console and execute the code.

It would be convenient to put in the language section, American English because, from my experience, some title's translation won't be on letterboxd.


Here is the script:

```
async function scrollAndExtract() {
    let previousHeight = document.body.scrollHeight; // Store the previous height of the page
    let titles = []; // Initialize an array to store movie titles and years

    // Function to scroll to the bottom of the page
    const scrollToBottom = () => {
        window.scrollTo(0, document.body.scrollHeight);
    };

    // Repeat until there is no more content to load
    while (true) {
        scrollToBottom(); // Scroll down
        await new Promise(resolve => setTimeout(resolve, 2000)); // Wait for 2 seconds to allow content to load

        // Get the new height of the page after scrolling
        let newHeight = document.body.scrollHeight;

        // If the height hasn't changed, we've reached the end of the content
        if (newHeight === previousHeight) {
            break; // Exit the loop if no more content is found
        }

        previousHeight = newHeight; // Update the previous height for the next iteration
    }

    // After scrolling, extract the movie titles and years
    let items = Array.from(document.querySelectorAll('h2.title-card-heading'));

    titles = items.map(item => {
        let titleWithYear = item.innerText.trim(); // Get the full title text, which includes the year
        let yearMatch = titleWithYear.match(/\((\d{4})\)$/); // Extract the year using regex, matching (YYYY) at the end
        let year = yearMatch ? yearMatch[1] : ""; // Extract the year or set to an empty string if not found
        let title = titleWithYear.replace(/\s*\(\d{4}\)$/, ""); // Remove the year from the title
        return { Title: title, Year: year }; // Return an object containing the cleaned title and year
    });

    // Create the CSV content in the required format: "Title","Year"
    let csvContent = "Title,Year\n" + 
        titles.map(e => `"${e.Title}","${e.Year}"`).join("\n"); // Ensure titles and years are in quotes
    let encodedCsvContent = encodeURIComponent(csvContent); // Encode the CSV content for the link
    let encodedUri = "data:text/csv;charset=utf-8," + encodedCsvContent; // Create the URI with the encoding
    let link = document.createElement("a"); // Create a link element
    link.setAttribute("href", encodedUri); // Set the href to the encoded CSV content
    link.setAttribute("download", "justwatch_watchlist.csv"); // Set the download filename
    document.body.appendChild(link); // Append the link to the body
    link.click(); // Trigger a click to download the CSV
    
    console.log("CSV generated successfully."); // Log a success message
}

// Execute the function
scrollAndExtract();


```
It will drop a csv, upload it on https://letterboxd.com/import/ et voila!
