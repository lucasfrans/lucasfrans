<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>PDP-11 Portfolio</title>
    <style>
        body, html {
            margin: 0;
            padding: 0;
            height: 100%;
            background-color: #000000;
            font-family: 'Lucida', 'Lucida Console', monospace;
            color: #FFB000;
            overflow: hidden; /* Keep body overflow hidden */
            font-size: 16px;
        }
        
        .screen {
            position: relative;
            width: 100%;
            height: 100%;
            padding: 40px;
            box-sizing: border-box;
            display: flex;
            flex-direction: column;
            justify-content: flex-start;
            align-items: flex-start;
            text-align: left;
            overflow: auto; /* Changed to auto to allow scrolling within the screen */
            cursor: text; /* Indicate that the area is generally for text input */
        }
        
        .scanlines {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(
                to bottom,
                rgba(255, 255, 255, 0.03) 50%,
                rgba(0, 0, 0, 0.1) 50%
            );
            background-size: 100% 4px;
            pointer-events: none;
            z-index: 10;
            opacity: 0.3;
        }
        
        #terminal {
            width: 100%;
            display: flex;
            flex-direction: column;
            align-items: flex-start;
            margin-bottom: 20px; /* Keep margin for spacing */
            text-align: left;
            max-width: 100%;
            overflow-x: hidden; /* Prevent terminal itself from overflowing horizontally */
            font-size: 16px;
        }
        
        pre {
            margin: 0;
            font-family: 'Lucida', 'Lucida Console', monospace;
            /* Default white-space will be set by JS, but 'pre-wrap' is often better */
             white-space: pre-wrap; /* Set a default that wraps */
            overflow: visible; /* Let wrapping handle overflow */
            max-width: 100%;
            font-size: 16px;
             cursor: text; /* Ensure text cursor over output too */
        }
        
        .input-line {
            display: flex;
            width: 100%;
            justify-content: flex-start;
            margin: 0;
            text-align: left;
            min-height: 1.2em; /* Ensure line takes up space */
        }
        
        .prompt {
            margin-right: 8px;
            color: #FFB000;
            white-space: nowrap; /* Prevent prompt from wrapping */
             cursor: text; /* Ensure text cursor over prompt */
        }
        
        #command-input {
            background: transparent;
            border: none;
            color: #FFC966; /* Lighter shade for command text being typed */
            font-family: 'Lucida', 'Lucida Console', monospace;
            font-size: 16px;
            outline: none;
            caret-color: #FFC966; /* Matching cursor color */
            text-align: left;
            padding: 0;
            width: auto; /* Let it take space */
            display: inline;
            flex-grow: 1; /* Allow it to fill remaining space */
            min-width: 10px; /* Ensure it's clickable */
             cursor: text; /* Explicitly set text cursor */
        }
        
        @keyframes blink {
            0%, 100% { opacity: 1; }
            50% { opacity: 0; }
        }
        
        .hidden {
            display: none;
        }
        
        .boot-sequence {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            display: flex;
            flex-direction: column;
            justify-content: flex-start;
            align-items: flex-start;
            padding: 40px;
            box-sizing: border-box;
            z-index: 20;
            background-color: #000000;
            text-align: left;
            font-family: 'Lucida', 'Lucida Console', monospace;
            font-size: 16px;
             overflow: hidden; /* Prevent boot sequence itself from scrolling */
        }
        
         .boot-sequence pre {
             margin: 0;
             white-space: pre;
             font-family: 'Lucida', 'Lucida Console', monospace;
        }
         .boot-sequence div {
            font-family: 'Lucida', 'Lucida Console', monospace;
        }
    </style>
</head>
<body>
    <div class="boot-sequence" id="boot-sequence">
        </div>

            <div class="screen">
        <div class="scanlines"></div>
        <div id="terminal">
            </div>
    </div>

    <script>
        // Boot sequence messages
        const bootMessages = [
            "PDP-11/70 BOOT SEQUENCE INITIATED",
            "MEMORY CHECK... OK",
            "16K WORDS MEMORY DETECTED",
            "LOADING BASIC MONITOR...",
            "INITIALIZING SYSTEM RESOURCES",
            "MOUNTING FILE SYSTEM",
            "CHECKING DIRECTORIES",
            "LOADING PORTFOLIO OS V1.0",
            "SYSTEM READY"
        ];
        
        // Available commands
        const commands = {
            help: "Displays available commands",
            ls: "Lists contents of current directory",
            cd: "Change directory (currently only root is available)",
            cat: "Display file contents",
            clear: "Clear the terminal",
            about: "Display information about me",
            education: "View my education history",
            experience: "View my work experience",
            social: "View social media links",
            contact: "How to contact me"
        };
        
        // Directory structure (simplified for demo)
        const fileSystem = {
            root: {
                about: "I'm Lucas Fransen, and right now I'm helping run the internet @ AWS, deploying the networking infrastructure that powers things like high-performance computing to support machine learning – also other neat stuff. Beyond that current side quest, I'm a pilot who likes to putt around in my little airplane, and who has a genuine fascination for all things tech especially infrastructure. While I'm still connecting the dots in my career, I thrive on the challenge of figuring things out and bringing a versatile approach to the table.",
                education: "- Columbia Basin College: Associate of Arts and Sciences - AAS, Computer Science (Sep 2022-Jul 2024, ~3.5 GPA)\n- Hanford High School: High School Diploma",
                experience: "- Global Network Delivery - Scaling IV at AWS (Dec 2024-Present)\n- Infrastructure Delivery Scaling III at Amazon (Jun 2024-Dec 2024)\n- Board Level Repair at Lucas' Board Repair (Feb 2020-Dec 2024)\n- Barista at Wake Up Call Coffee (Mar 2023-May 2024)\n- Game Master/IT at Atomic Escape Rooms (Mar 2021-Jun 2021)",
                social: "- LinkedIn: linkedin.com/in/lucas-fransen\n- Instagram: https://www.instagram.com/lucasfranssn/",
                contact: "Email: lucas.fransen1@gmail.com"
            }
        };
        
        // Current directory
        let currentDirectory = "root";
        
        // Boot sequence function
        async function bootSequence() {
            const bootElement = document.getElementById('boot-sequence');
            const screenElement = document.querySelector('.screen'); // Get screen for later use
            screenElement.classList.add('hidden'); // Hide main screen during boot
            
            // Display each boot message with delay - faster speed for 4-5 second total boot sequence
            for (const message of bootMessages) {
                const messageElement = document.createElement('div');
                messageElement.textContent = "";
                bootElement.appendChild(messageElement);
                
                // Type out the message faster
                for (const char of message) {
                    messageElement.textContent += char;
                    await sleep(10); // Faster typing
                }
                
                await sleep(200); // Shorter delay between messages
            }
            
            // Show a "loading" progress bar
            const progressContainer = document.createElement('div');
            progressContainer.style.marginTop = "10px";
            progressContainer.innerHTML = "LOADING SYSTEM [           ]"; // Adjusted spaces for 11 steps (0-10)
            bootElement.appendChild(progressContainer);
            
            // Fill the progress bar faster
            const progressText = progressContainer.innerHTML;
            for (let i = 0; i <= 10; i++) {
                 // Correctly replace space with '=' based on index i
                let newProgress = progressText.substring(0, 14 + i) + "=" + progressText.substring(15 + i);
                progressContainer.innerHTML = newProgress;
                await sleep(100); // Faster progress
            }
            
            await sleep(300); // Shorter final delay
            
            // Hide boot sequence and show terminal
            bootElement.style.opacity = 0;
            bootElement.style.transition = "opacity 0.5s";
            
            setTimeout(() => {
                bootElement.classList.add('hidden');
                screenElement.classList.remove('hidden'); // Show main screen
                
                // Add welcome message with ASCII art
                addToTerminal("", false); // Add blank line before art
                const asciiArt = `
+----------------------------------------------------------------------+
|  ____   ___  ____ _____ _____ ___  _     ___ ___    ___  ____         |
| |  _ \\ / _ \\|  _ \\_   _|  ___/ _ \\| |    |_ _/ _ \\  / _ \\/ ___|        |
| | |_) | | | | |_) || | | |_ | | | | |     | | | | || | | \\___ \\        |
| |  __/| |_| |  _ < | | |  _|| |_| | |___  | | |_| || |_| |___) |       |
| |_|    \\___/|_| \\_\\|_| |_|   \\___/|_____|___\\___/  \\___/|____/        |
|                                                                      |
+----------------------------------------------------------------------+
                     [ VERSION 1.0 ]
`; // Note: No extra newline needed after backtick if there's one before it in the string

                // *** FIXED HERE: Removed .trim() ***
                addToTerminal(asciiArt, false, false, true); // Pass true for preserveFormattingStrict=true
                
                addToTerminal("", false); // Add blank line after art
                
                // Simulate running the ls command
                addToTerminal("ls", true);
                const dirContent = Object.keys(fileSystem[currentDirectory]).join('   '); // Add more space between file names
                addToTerminal(dirContent || "NO FILES FOUND", false);
                addToTerminal("", false);
                
                // Simulate running the cat about command
                addToTerminal("cat about", true);
                if (fileSystem[currentDirectory]['about']) {
                    addToTerminal(fileSystem[currentDirectory]['about'], false);
                }
                addToTerminal("", false);
                
                // Add tip
                addToTerminal("TIP: TYPE 'HELP' TO SEE ALL AVAILABLE COMMANDS", false);
                addToTerminal("", false);
                
                // Create the command input directly in the terminal
                initializeCommandInput(); 

                // Add click listener to the screen to refocus input
                // const screenElement = document.querySelector('.screen');  <- Already defined above
                screenElement.addEventListener('click', function(event) {
                    const commandInput = document.getElementById('command-input');
                    // If the input exists and the click was *not* directly on the input itself
                    if (commandInput && event.target !== commandInput) {
                        commandInput.focus();
                    }
                });

            }, 500); // Wait for fade out
        }
        
        // Helper function for sleep/delay
        function sleep(ms) {
            return new Promise(resolve => setTimeout(resolve, ms));
        }
        
        // Initialize the command input directly in the terminal
        function initializeCommandInput() {
            const terminal = document.getElementById('terminal');
            // Check if an input line already exists and remove it if so
            const existingInputLine = terminal.querySelector('.input-line:has(#command-input)');
            if (existingInputLine) {
                terminal.removeChild(existingInputLine);
            }

            const inputLine = document.createElement('div');
            inputLine.className = 'input-line';
            
            const prompt = document.createElement('span');
            prompt.className = 'prompt';
            prompt.textContent = '~$';
            
            const input = document.createElement('input');
            input.id = 'command-input';
            input.type = 'text';
            input.autocomplete = 'off';
            input.autocorrect = 'off';
            input.autocapitalize = 'off';
            input.spellcheck = false;
            
            inputLine.appendChild(prompt);
            inputLine.appendChild(input);
            terminal.appendChild(inputLine);
            
            input.focus(); // Set focus when the input line is created
            
            // Add event listener to the new input
            input.addEventListener('keydown', function(e) {
                if (e.key === 'Enter') {
                    const command = this.value;
                    
                    // Remove current input line before processing
                    terminal.removeChild(inputLine); 
                    
                    // Add command to terminal history (with prefix)
                    addToTerminal(command, true);
                    
                    // Process command
                    processCommand(command);
                    
                    // Create new input line AFTER processing output
                    initializeCommandInput(); 
                    
                    // Scroll to bottom after command processing and new input line creation
                     const screen = document.querySelector('.screen'); 
                     screen.scrollTop = screen.scrollHeight;

                } else if (e.key === 'Tab') {
                    // Prevent default behavior (tab moving focus)
                    e.preventDefault();
                    
                    // Get the current input value
                    const currentInput = this.value.toLowerCase().trim().split(' ')[0]; // Autocomplete only the command part
                    
                    if (currentInput) {
                        // Check for command completion
                        const possibleCommands = Object.keys(commands).filter(cmd => 
                            cmd.startsWith(currentInput)
                        );
                        
                        if (possibleCommands.length === 1) {
                            // Only one match, complete it
                            this.value = possibleCommands[0] + ' '; // Add space after completion
                        } else if (possibleCommands.length > 1) {
                            // Multiple matches, show them to the user
                            // Remove current input line temporarily to show completions
                            const currentInputValue = this.value; // Save current value
                            terminal.removeChild(inputLine);
                            
                            // Show the command typed so far and possible completions
                            addToTerminal(currentInputValue, true); // Show what was typed
                            addToTerminal("Possible commands: " + possibleCommands.join("  "), false); // Show options
                            addToTerminal("", false); // Add a blank line
                            
                            // Re-create the input line with the original partial input
                            initializeCommandInput();
                            const newInput = document.getElementById('command-input');
                             if(newInput) {
                                 newInput.value = currentInputValue; // Restore partial input
                                 newInput.focus(); // Ensure focus is restored
                             }

                        }
                    }
                }
            });
        }
        
        // Add text to terminal
        // Added preserveFormattingStrict flag for ASCII art
        function addToTerminal(text, isCommand = true, useHTML = false, preserveFormattingStrict = false) {
            const terminal = document.getElementById('terminal');
             // Find the current input line if it exists, to insert output before it
            const inputLine = terminal.querySelector('.input-line:has(#command-input)');
            
            const output = document.createElement('div');
            
            if (isCommand) {
                output.className = 'input-line'; // Use same class for consistency, but it's output
                // Keep prompt the original color, make command text lighter
                output.innerHTML = `<span class="prompt">~$</span> <span style="color: #FFC966;">${text}</span>`;
            } else if (useHTML) {
                // Use innerHTML for help text that contains HTML
                output.style.textAlign = 'left';
                output.style.color = '#FFFFFF'; // White output text
                output.style.fontFamily = "'Lucida', 'Lucida Console', monospace";
                output.style.fontSize = '16px';
                output.innerHTML = text; // Directly set HTML
            } else {
                // Standard text output
                output.style.textAlign = 'left';
                output.style.color = '#FFFFFF'; // White output text
                
                // Use pre element for formatting control
                const pre = document.createElement('pre');
                pre.style.margin = '0';
                pre.style.fontFamily = "'Lucida', 'Lucida Console', monospace";
                
                // Use 'pre' for strict formatting (ASCII art), 'pre-wrap' otherwise
                pre.style.whiteSpace = preserveFormattingStrict ? 'pre' : 'pre-wrap'; 
                
                pre.style.fontSize = '16px';
                pre.textContent = text; // Set text content
                output.appendChild(pre); // Add pre to the output div
            }
            
            // Insert the output before the input line, or at the end if no input line exists
            if (inputLine) {
                 terminal.insertBefore(output, inputLine);
            } else {
                 terminal.appendChild(output);
            }
        }
        
        // Process commands
        function processCommand(command) {
            const cmd = command.trim().toLowerCase();
            const args = cmd.split(' ');
            const commandName = args[0];
            
            addToTerminal("", false); // Add space before command output
            
            switch(commandName) {
                case 'help':
                    let helpText = "AVAILABLE COMMANDS:";
                    Object.keys(commands).forEach(key => {
                        // Format each line for better readability
                        helpText += `<br><span style="color: #FFC966; display: inline-block; width: 100px;">${key}</span>: ${commands[key]}`;
                    });
                    addToTerminal(helpText, false, true); // Use HTML for formatting
                    break;
                    
                case 'clear':
                     const terminal = document.getElementById('terminal');
                     terminal.innerHTML = ''; 
                    break;
                    
                case 'ls':
                    const dirContent = Object.keys(fileSystem[currentDirectory]).join('   '); // More space
                    addToTerminal(dirContent || "NO FILES FOUND", false);
                    break;
                
                case 'cd':
                     if (args.length < 2 || args[1] === '/' || args[1] === 'root' || args[1] === '.' || args[1] === '..') {
                         currentDirectory = "root"; 
                     } else {
                        addToTerminal(`ERROR: DIRECTORY NOT FOUND: ${args[1].toUpperCase()}. ONLY ROOT ('/') IS AVAILABLE.`, false);
                     }
                    break;
                    
                case 'cat':
                    if (args.length < 2) {
                        addToTerminal("USAGE: CAT <FILENAME>", false);
                    } else {
                        const fileName = args[1];
                        if (fileSystem[currentDirectory][fileName]) {
                            addToTerminal(fileSystem[currentDirectory][fileName], false);
                        } else {
                            addToTerminal(`FILE NOT FOUND: ${fileName}`, false);
                        }
                    }
                    break;
                    
                case 'about':
                case 'education':
                case 'experience':
                case 'social':
                case 'contact':
                    if (fileSystem[currentDirectory][commandName]) {
                        addToTerminal(fileSystem[currentDirectory][commandName], false);
                    } else {
                        addToTerminal(`ERROR: CONTENT FOR ${commandName} NOT FOUND.`, false); 
                    }
                    break;
                    
                case '':
                    break;
                    
                default:
                    addToTerminal(`COMMAND NOT FOUND: ${commandName}. TYPE 'HELP' FOR OPTIONS.`, false);
            }
            
            addToTerminal("", false); // Add space after command output
        }
        
        // Start the boot sequence when page loads
        window.onload = bootSequence;
    </script>
</body>
</html>
