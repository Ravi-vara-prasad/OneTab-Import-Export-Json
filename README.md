# OneTab-Import-Export-Json
<details>
  <summary>Export Script</summary>

  ```
(async function exportOneTab() {
  const dbName = 'onetab';
  const request = indexedDB.open(dbName);

  request.onsuccess = async (event) => {
    const db = event.target.result;
    const exportData = {};
    const storeNames = Array.from(db.objectStoreNames);

    console.log(`Found object stores: ${storeNames.join(', ')}. Extracting...`);

    for (const storeName of storeNames) {
      const transaction = db.transaction(storeName, 'readonly');
      const store = transaction.objectStore(storeName);
      const allRecords = await new Promise((resolve) => {
        const req = store.getAll();
        req.onsuccess = () => resolve(req.result);
      });
      exportData[storeName] = allRecords;
    }

    const jsonString = JSON.stringify(exportData, null, 2);
    const blob = new Blob([jsonString], { type: 'application/json' });
    const url = URL.createObjectURL(blob);
    
    const a = document.createElement('a');
    a.href = url;
    a.download = 'onetab-full-backup.json';
    document.body.appendChild(a);
    a.click();
    document.body.removeChild(a);
    URL.revokeObjectURL(url);
    
    console.log("✅ Backup complete! Your JSON file should be downloading now.");
  };

  request.onerror = (event) => {
    console.error("Database error: ", event.target.error);
  };
})();


  ```
  
</details>

<details>
  <summary> Import Script</summary>

  ```
(function importOneTab() {
  const input = document.createElement('input');
  input.type = 'file';
  input.accept = 'application/json';

  input.onchange = (event) => {
    const file = event.target.files[0];
    if (!file) return;

    const reader = new FileReader();
    reader.onload = async (e) => {
      try {
        const data = JSON.parse(e.target.result);
        const request = indexedDB.open('onetab');

        request.onsuccess = (ev) => {
          const db = ev.target.result;
          const storeNames = Object.keys(data).filter(name => db.objectStoreNames.contains(name));
          
          if (storeNames.length === 0) {
             console.error("Could not find the correct OneTab databases.");
             return;
          }

          const transaction = db.transaction(storeNames, 'readwrite');
          
          transaction.oncomplete = () => {
            console.log("✅ Import complete! Refresh the OneTab page to see your tabs and groups.");
          };

          transaction.onerror = (err) => {
            console.error("Database write error:", err);
          };

          storeNames.forEach(storeName => {
            const store = transaction.objectStore(storeName);
            data[storeName].forEach(item => {
              store.put(item); 
            });
          });
        };
      } catch (err) {
        console.error("Error parsing the JSON file:", err);
      }
    };
    reader.readAsText(file);
  };

  input.click();
})();
<details>












  ```
</details>
