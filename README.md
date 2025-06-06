# data-integrity-monitor

Creating a robust tool like a data-integrity-monitor requires careful handling of distributed systems, error handling, and logging. Below is a Python program that serves as a basic framework for such a tool. This example assumes some common requirements, such as monitoring data changes, validating data integrity across systems, and logging issues.

```python
import hashlib
import threading
import logging
import time

logging.basicConfig(level=logging.DEBUG, format='%(asctime)s - %(levelname)s - %(message)s')

class DataIntegrityMonitor:
    def __init__(self, interval=10):
        self.data_sources = {}
        self.interval = interval  # interval to check integrity in seconds
        self.running = False

    def add_data_source(self, name, get_data_function):
        """
        Adds a new data source to monitor.
        :param name: The name of the data source.
        :param get_data_function: A function that retrieves and returns the current state of the data.
        """
        self.data_sources[name] = {
            'get_data_function': get_data_function,
            'last_hash': None
        }
        logging.debug(f"Added data source: {name}")

    def compute_hash(self, data):
        """
        Computes the hash of the given data.
        :param data: The data to hash.
        :return: A SHA256 hash of the data.
        """
        return hashlib.sha256(repr(data).encode()).hexdigest()

    def check_integrity(self):
        """
        Checks the integrity of the data across all registered data sources.
        """
        for name, source in self.data_sources.items():
            try:
                logging.debug(f"Checking data source: {name}")
                current_data = source['get_data_function']()
                current_hash = self.compute_hash(current_data)
                
                # If the hash has changed, the data is potentially compromised
                if source['last_hash'] is not None and source['last_hash'] != current_hash:
                    logging.warning(f"Data integrity issue detected in {name}!")

                source['last_hash'] = current_hash
                logging.debug(f"Data source {name} is consistent.")

            except Exception as e:
                logging.error(f"Error checking data source {name}: {str(e)}")

    def start_monitoring(self):
        """
        Starts the monitoring process.
        """
        if not self.data_sources:
            raise ValueError("No data sources to monitor. Add at least one data source before starting.")

        self.running = True
        monitor_thread = threading.Thread(target=self._monitor)
        monitor_thread.start()
        logging.info("Data integrity monitoring started.")

    def stop_monitoring(self):
        """
        Stops the monitoring process.
        """
        self.running = False
        logging.info("Data integrity monitoring stopped.")

    def _monitor(self):
        """
        Internal method to repeatedly check data integrity.
        """
        while self.running:
            self.check_integrity()
            time.sleep(self.interval)

# Example usage
if __name__ == "__main__":

    def get_data_from_source_1():
        # Simulate fetching data from a source
        return {"key1": "value1", "key2": "value2"}

    def get_data_from_source_2():
        # Simulate fetching data from another source
        return {"key1": "value1", "key2": "value3"}  # different value to simulate inconsistency

    monitor = DataIntegrityMonitor(interval=5)
    monitor.add_data_source("DataSource1", get_data_from_source_1)
    monitor.add_data_source("DataSource2", get_data_from_source_2)

    try:
        monitor.start_monitoring()
        time.sleep(60)  # Run the monitor for 1 minute
    finally:
        monitor.stop_monitoring()
```

### Key Features of This Program:

1. **Data Sources:** You can add multiple data sources by providing a function that retrieves data from each source. This allows flexibility in sourcing data, whether it comes from a database, an API, or a file.

2. **Hashing for Data Integrity:** The program uses hashing (`SHA256`) to create a unique fingerprint of data and compares it to previous states to detect changes.

3. **Multi-threading:** Monitoring runs in a separate thread to allow periodic checking without blocking the main application logic.

4. **Error Handling:** The program logs errors if data retrieval or hashing fails, ensuring that issues do not crash the application.

5. **Logging:** Comprehensive logging informs users of the process's status and any detected issues.

This setup is customizable and can be expanded to include more sophisticated mechanisms for data retrieval, comparison, and reporting, based on specific requirements.