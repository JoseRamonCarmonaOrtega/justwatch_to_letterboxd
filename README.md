# JustWatch's watchlist to letterboxd
IMPORTANT: You need to be on https://www.justwatch.com/us/lists/my-lists and display the movies in list format as you can see on this photo

![image](https://github.com/user-attachments/assets/5cbc86e6-56d2-4bb8-b4d8-c8b863e5234b)

Then, open the console and execute the code.

It would be convenient to put in the language section, American English because, from my experience, some title's translation won't be on letterboxd.


Here is the script:

```
async function scrollAndExtract() {
    let previousHeight = 0;
    let retries = 0;
    const MAX_RETRIES = 3; // Prevents the script from stopping prematurely due to network lag

    console.log("Starting to scroll...");

    // 1. Scroll to the bottom with a built-in retry mechanism
    while (retries < MAX_RETRIES) {
        window.scrollTo(0, document.body.scrollHeight);
        await new Promise(resolve => setTimeout(resolve, 1500)); 

        const newHeight = document.body.scrollHeight;
        if (newHeight === previousHeight) {
            retries++; // If no new content loaded, increment the retry counter
        } else {
            retries = 0; // If content successfully loaded, reset the counter
            previousHeight = newHeight;
        }
    }

    console.log("Extracting titles...");

    // 2. DOM Extraction and data mapping in a single pass
    const items = document.querySelectorAll('h2.title-card-heading');
    const titles = Array.from(items, item => {
        const text = item.innerText.trim();
        const yearMatch = text.match(/\((\d{4})\)$/);
        const year = yearMatch ? yearMatch[1] : "";
        
        // Remove the year from the title string and escape double quotes for valid CSV formatting
        const title = text.replace(/\s*\(\d{4}\)$/, "").replace(/"/g, '""');
        
        return `"${title}","${year}"`;
    });

    if (titles.length === 0) {
        console.warn("No movies found. Please make sure you are using the list view format.");
        return;
    }

    // 3. CSV Generation using Blob (Bypasses URL length limits for massive watchlists)
    const csvContent = "Title,Year\n" + titles.join("\n");
    
    // Inject BOM (Byte Order Mark) to ensure Excel and other parsers read special characters correctly
    const blob = new Blob([new Uint8Array([0xEF, 0xBB, 0xBF]), csvContent], { type: "text/csv;charset=utf-8;" });
    const url = URL.createObjectURL(blob);

    // 4. Trigger download and perform cleanup
    const link = document.createElement("a");
    link.href = url;
    link.download = "justwatch_watchlist.csv";
    document.body.appendChild(link);
    link.click();
    
    document.body.removeChild(link); // Keep the DOM clean
    URL.revokeObjectURL(url); // Free up browser memory

    console.log(`Success! CSV generated with ${titles.length} movies.`);
}

// Execute the extraction
scrollAndExtract();
```
It will drop a csv, upload it on https://letterboxd.com/import/ et voila!

Note: it also works for the rest of sections with the same display as my lists, I mean, 1/2 Lists, Watchlist, Seen, Liked, etc.
