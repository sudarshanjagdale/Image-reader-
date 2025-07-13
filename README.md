// Create or update a single persistent toast
function createOrUpdateToast(message) {
    let toast = document.getElementById('persistent-toast');
    if (!toast) {
        toast = document.createElement('div');
        toast.id = 'persistent-toast';
        Object.assign(toast.style, {
            position: 'fixed',
            top: '20px',
            right: '20px',
            zIndex: 9999,
            maxWidth: '400px',
            padding: '12px 20px',
            borderRadius: '8px',
            backgroundColor: 'rgba(0, 0, 0, 0.85)',
            color: 'white',
            fontSize: '14px',
            fontFamily: 'Segoe UI, Tahoma, Geneva, Verdana, sans-serif',
            boxShadow: '0px 0px 12px rgba(0,0,0,0.5)',
            userSelect: 'none',
            transition: 'background-color 0.3s ease'
        });
        document.body.appendChild(toast);
    }
    toast.textContent = message;
}

// CSV download function
function downloadCSV(data, filename = 'hs_training_data.csv') {
    const header = 'Document ID,Pages,Layout,Usage Rule,Source';
    const csvContent = data
        .map(row => row.map(cell => `"${cell}"`).join(','))
        .join('\n');
    const csv = `${header}\n${csvContent}`;
    const blob = new Blob([csv], { type: 'text/csv;charset=utf-8;' });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = filename;
    a.style.display = 'none';
    document.body.appendChild(a);
    a.click();
    document.body.removeChild(a);
    setTimeout(() => URL.revokeObjectURL(url), 100);
}

// Main scraping logic
async function extractFullTableWithPagination(cardSelector, headingText) {
    let maxPages = prompt("📄 How many pages to scrape?", "3");
    maxPages = parseInt(maxPages, 10);
    if (isNaN(maxPages) || maxPages < 1) {
        alert("❌ Invalid input. Using default 3 pages.");
        maxPages = 3;
    }

    let currentPage = 0;
    const capturedData = [];

    while (currentPage < maxPages) {
        createOrUpdateToast(`📄 Extracting page ${currentPage + 1}...`);

        // Wait for page data to load (40 sec recommended)
        await new Promise(resolve => setTimeout(resolve, 40000));

        const tableRows = document.querySelectorAll('table tbody tr');
        tableRows.forEach(row => {
            const cells = row.querySelectorAll('td');
            if (cells.length >= 6) {
                const rowData = [
                    cells[0]?.innerText.trim(), // Document ID
                    cells[1]?.innerText.trim(), // Pages
                    cells[2]?.innerText.trim(), // Layout
                    cells[3]?.innerText.trim(), // Usage Rule
                    cells[4]?.innerText.trim()  // Source
                ];
                capturedData.push(rowData);
            }
        });

        currentPage++;

        if (currentPage >= maxPages) break;

        const nextPageBtn = document.querySelector('button[aria-label="Next page (Page Down)"]');
        if (nextPageBtn && !nextPageBtn.disabled) {
            nextPageBtn.click();
        } else {
            console.warn("❌ Next button not found or disabled. Stopping.");
            createOrUpdateToast("❌ Next page link not found. Stopping.");
            break;
        }
    }

    createOrUpdateToast(`✅ Done! Downloading ${capturedData.length} records...`);
    downloadCSV(capturedData);
}

// Run
extractFullTableWithPagination('[data-component="Card"]', 'Excluded Training Data');