// Show toast message on screen
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
            padding: '12px 20px',
            borderRadius: '8px',
            backgroundColor: 'rgba(0, 0, 0, 0.85)',
            color: 'white',
            fontSize: '14px',
            fontFamily: 'Segoe UI, Tahoma, sans-serif',
            boxShadow: '0px 0px 12px rgba(0,0,0,0.5)'
        });
        document.body.appendChild(toast);
    }
    toast.textContent = message;
}

// Download data as CSV
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

// Main function
async function extractTableDataFromPages() {
    let maxPages = prompt("📄 How many pages to scrape?", "3");
    maxPages = parseInt(maxPages, 10);
    if (isNaN(maxPages) || maxPages < 1) {
        alert("❌ Invalid input. Using default 3 pages.");
        maxPages = 3;
    }

    let currentPage = 0;
    const capturedData = [];

    while (currentPage < maxPages) {
        createOrUpdateToast(`📄 Extracting Page ${currentPage + 1}...`);

        // Wait for page content to fully load
        await new Promise(resolve => setTimeout(resolve, 40000)); // 40 sec

        // Extract table data
        const rows = document.querySelectorAll('table tbody tr');
        rows.forEach(row => {
            const cells = row.querySelectorAll('td');
            if (cells.length >= 5) {
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

        // Try to go to next page
        const targetCard = document.querySelector('[data-component="Card"]');
        const nextPageLink = targetCard?.querySelector('[aria-label="Next page (Page Down)"]');

        if (nextPageLink) {
            nextPageLink.click();
        } else {
            const stopMsg = '⚠️ Next page link not found. Stopping.';
            console.warn(stopMsg);
            createOrUpdateToast(stopMsg);
            break;
        }

        currentPage++;
    }

    createOrUpdateToast(`✅ Done! Downloading ${capturedData.length} records...`);
    downloadCSV(capturedData);
}

// Run it
extractTableDataFromPages();