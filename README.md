# OneTab-Import-Export-Json

<details>
  <summary>Import Script</summary>
  
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
</details>
