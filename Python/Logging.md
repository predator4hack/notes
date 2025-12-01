
Setting a logger's level using `logger.setLevel(logging.INFO)` means that the logger will process all log messages that are at the **INFO level or higher in severity**.

## 1. 🚨 Logging Levels (The Five Standard Levels)

	The core of logging is understanding and using the five standard severity levels. By default, the `logging` module processes messages at **WARNING** level and above.
	

| **Level**    | **Numeric Value** | **When to Use**                                                                                                                                                          |
| ------------ | ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **DEBUG**    | 10                | Detailed information, typically of interest only when diagnosing problems.                                                                                               |
| **INFO**     | 20                | Confirmation that things are working as expected.                                                                                                                        |
| **WARNING7** | 30                | An indication that something unexpected happened, or indicative of some problem in the near9 future (e.g., 'disk space low'). The software is still working as expected. |
| **ERROR**    | 40                | Due to a more serious problem, the software has not been able to perform some function.                                                                                  |
| **CRITICAL** | 50                | A serious error, indicating that the program itself may be unable to continue running.                                                                                   |

A good developer should use the appropriate level: don't log an application crash as a **WARNING**.

---

## 3. 📝 The Core Components

The `logging` module uses a modular approach with four main components.

### A. **Loggers**

- This is the entry point. Loggers expose the five severity methods (`.debug()`, `.info()`, etc.) that you call in your application code.
- They are organized in a **hierarchical namespace** (like Python modules: `my_app.db`, `my_app.web`).
- **The Root Logger:** The top-level logger from which all others inherit. If you don't specify a logger name, you use the root logger.
- **Best Practice:** Always get your logger using `logger = logging.getLogger(__name__)`. This automatically names the logger after the current module, making it easy to trace where a log message came from.

### B. **Handlers**

- Handlers determine **where** the log message goes (the destination).
- **Common Handlers:**
    - `StreamHandler`: Sends messages to streams (like `sys.stdout` or `sys.stderr`). Used for console output.
    - `FileHandler`: Sends messages to a disk file.
    - `RotatingFileHandler`: Writes to a file, but automatically rolls over to a new file when the current one reaches a certain size.
    - `TimedRotatingFileHandler`: Writes to a file, and automatically rolls over at certain time intervals (e.g., daily).
    - `HTTPHandler`, `SMTPHandler`, etc.

### C. **Formatters**

- Formatters specify the **layout** of the log record. They use a standard string formatting style or Python's `logging` style attributes.
- A good format includes a **timestamp**, **level name**, **logger name**, and the **message**.
- **Example Format String:**
    ```
    '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
    ```

### D. **Filters**

- Filters provide an extra level of control over which log records are passed. They can be attached to **Loggers** or **Handlers**. They are rarely needed but allow for advanced scenarios (e.g., only logging messages from specific users).

---

## 4. 💻 Basic Implementation and Configuration

A good developer moves beyond the simplistic default configuration.
### Basic Setup (for console output)
For simple applications, you can configure the root logger:

```python
import logging

logging.basicConfig(
    level=logging.INFO, # Sets the threshold for the root logger
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)

# Get a logger named after the module
logger = logging.getLogger(__name__)

logger.info("Application starting up.")
logger.debug("This message won't show, as the level is INFO.")
```

### Advanced Configuration (Recommended)

For production applications, it's highly recommended to use a **configuration dictionary** or a **configuration file** (like YAML or INI) to set up all loggers, handlers, and formatters in one place.

```python
import logging.config

LOGGING_CONFIG = {
    'version': 1,
    'disable_existing_loggers': False, # Important: Don't disable existing loggers
    
    'formatters': {
        'standard': {
            'format': '%(asctime)s [%(levelname)s] %(name)s: %(message)s'
        },
    },
    
    'handlers': {
        'console': {
            'class': 'logging.StreamHandler',
            'formatter': 'standard',
            'level': 'INFO',
        },
        'file': {
            'class': 'logging.handlers.RotatingFileHandler',
            'filename': 'app.log',
            'maxBytes': 1024 * 1024 * 5, # 5 MB
            'backupCount': 5,
            'formatter': 'standard',
            'level': 'DEBUG',
        },
    },
    
    'loggers': {
        # 'my_app' and its children (e.g., 'my_app.db')
        'my_app': { 
            'handlers': ['console', 'file'],
            'level': 'DEBUG', # Loggers should usually be set to DEBUG
            'propagate': False # Stop propagation to the root logger
        },
        # For third-party libraries (often too chatty)
        'requests': { 
            'handlers': ['console'],
            'level': 'WARNING',
            'propagate': False
        }
    }
}

logging.config.dictConfig(LOGGING_CONFIG)

# Get the configured logger
app_logger = logging.getLogger('my_app')
```

---

## 5. 🌟 Best Practices for a Good Developer

- **Use Exception Logging:** When catching exceptions, use `logger.exception()` instead of `logger.error()`. The `exception()` method is only callable from an exception handler and will automatically include the current exception trace (the stack trace) in the log message.
    ```python
    try:
        1 / 0
    except ZeroDivisionError as e:
        logger.exception("An error occurred during calculation.") 
        # Output includes full traceback!
    ```
    
- **Avoid String Concatenation:** Do **not** use f-strings or standard string concatenation _before_ calling the logging method, especially for `DEBUG` level messages.
    - **Bad:** `logger.debug("The result is: " + str(expensive_calculation()))`
    - **Good:** `logger.debug("The result is: %s", expensive_calculation())`
    - The **Good** way uses a format string and arguments. The `logging` module will only perform the **expensive calculation** (and string formatting) if the log message level is high enough to be processed. This is a critical performance detail.
        
- **Propagate Control:** Be aware of the `propagate` attribute on loggers. By default, log messages propagate up the hierarchy to parent loggers (and eventually the root logger). Setting it to `False` (`'propagate': False`) stops this behavior, giving you total control over where a message is handled.