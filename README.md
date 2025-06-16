(async function autoClickCheckBoxesForSelectedDocs() {
    // ✅ Replace this with your actual document ID list
    const documentIdsToClick = [
        '124979', '122573', '121868', '121913', '121912', '121921',
        '121917', '121353', '121759', '122767', '122325', '125231',
        '122314', '122103', '122757'
    ];

    // ✅ Utility: Toast Message for Status
    function createOrUpdateToast(message) {
        let toast = document.getElementById("persistent-toast");
        if (!toast) {
            toast = document.createElement("div");
            toast.id = "persistent-toast";
            Object.assign(toast.style, {
                position: 'fixed',
                top: "20px",
                right: "20px",
                zIndex: 9999,
                maxWidth: "320px",
                padding: "12px 20px",
                borderRadius: "8px",
                backgroundColor: "rgba(0, 0, 0, 0.85)",
                color: "white",
                fontSize: "15px",
                fontFamily: "Segoe UI, Tahoma, Geneva, Verdana, sans-serif",
                boxShadow: "0 4px 12px rgba(0,0,0,0.3)",
                userSelect: "none"
            });
            document.body.appendChild(toast);
        }
        toast.textContent = message;
    }

    // ✅ Ask user how many pages to loop through
    let maxPages = prompt('How many pages to scan?', '3');
    maxPages = parseInt(maxPages, 10);
    if (isNaN(maxPages) || maxPages < 1) {
        alert('Invalid number. Using default 3.');
        maxPages = 3;
    }

    let currentPage = 0;

    while (currentPage < maxPages) {
        createOrUpdateToast(`🔍 Scanning page ${currentPage + 1} of ${maxPages}...`);
        console.log(`Scanning page ${currentPage + 1}...`);

        // ⏳ Wait for table content to render
        await new Promise(resolve => setTimeout(resolve, 3000));

        // ✅ Loop through all rows, match IDs, and click checkboxes
        const rows = document.querySelectorAll('tr');
        let clickedCount = 0;

        rows.forEach(row => {
            const text = row.innerText || "";
            const docIdMatch = documentIdsToClick.find(id => text.includes(id));
            if (docIdMatch) {
                const checkbox = row.querySelector('input[type="checkbox"]');
                if (checkbox && !checkbox.checked) {
                    checkbox.click();
                    clickedCount++;
                    console.log(`✔ Clicked checkbox for Document ID: ${docIdMatch}`);
                }
            }
        });

        createOrUpdateToast(`✅ Page ${currentPage + 1}: Clicked ${clickedCount} checkboxes`);

        // 👉 Move to next page
        const nextBtn = document.querySelector('button[aria-label="Next page"]');
        if (nextBtn && !nextBtn.disabled) {
            nextBtn.click();
            currentPage++;
        } else {
            createOrUpdateToast("✅ No more pages or 'Next' button not found.");
            break;
        }
    }

    createOrUpdateToast("🎉 Done! All matching document checkboxes clicked.");
})();