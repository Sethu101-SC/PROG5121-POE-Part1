PROG5121w_FinalPOE
/*
 * Click nbfs://nbhost/SystemFileSystem/Templates/Licenses/license-default.txt to change this license
 */

package com.mycompany.prog5121w_finalpoe;

import javax.swing.JOptionPane;
import java.util.Random;
import java.io.FileWriter;
import java.io.IOException;
import java.util.ArrayList;
//Imports that read the stored JSON file and break it down
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.util.regex.Matcher;
import java.util.regex.Pattern;
//Imports used to display the longest stored message.
import java.awt.Dimension;
import javax.swing.JScrollPane;
import javax.swing.JTextArea;

/*
 * @author sethu
 */
public class PROG5121w_FinalPOE {
    //Declarations
    static String registeredUsername;
    static String registeredPassword;
    static String registeredCellNumber;
    static int totalMessagesSent = 0;
    static int messageNumber = 0;
    static final String cellNumber = "^\\+27[6-8]\\d{8}$";
    static ArrayList<String> sentMessages = new ArrayList<>();  //Conatins all the messages sent
    //Additional arrays
    static ArrayList<String> disregardedMessages = new ArrayList<>();  //Contains all messages that were disregarded 
    static ArrayList<String> storedMessagesFile = new ArrayList<>();  //Contains stored messages read from the JSON file
    static ArrayList<String> messageHashes = new ArrayList<>();  //conatins all message hashes
    static ArrayList<String> messageIDs = new ArrayList<>();    //Contains all message IDs  
    //Arrays that hold other details of each store message from JSON file.
    static ArrayList<String> storedMessageIDs = new ArrayList<>(); 
    static ArrayList<String> storedMessageHashes = new ArrayList<>();
    static ArrayList<String> storedRecipients = new ArrayList<>();
    
    //Welcome page
    public static void main(String[] args) {
        JOptionPane.showMessageDialog(null, "Welcome to QuickChat...");
        registerUser();

        if (loginUser()) {
            JOptionPane.showMessageDialog(null, "Login successful. Redirecting to QuickChat...");
            runMessageSystem();
        } else {
            JOptionPane.showMessageDialog(null, "Login failed. Exiting app.");
        }
    }
    
    //Shows the input dialog that lets the user register their details fro the application.
    public static String prompt(String message) {
        String input = JOptionPane.showInputDialog(message);
        if (input == null) {
            JOptionPane.showMessageDialog(null, "Cancelled. Exiting app.");
            System.exit(0);
        }
        return input;
    }
    
    //Registers the user information.
    public static void registerUser() {
        boolean valid = false;

        while (!valid) {
            registeredUsername = prompt("Create a username (must contain _ and maximum 5 characters.):");
            if (!(registeredUsername.contains("_") && registeredUsername.length() <= 5)) {
                JOptionPane.showMessageDialog(null, "Invalid username.");
                continue;
            }
            registeredPassword = prompt("Create a password (minimum 8 characters long, atleast 1 capital letter, 1 number and 1 special character):");
            if (!registeredPassword.matches("^(?=.*[A-Z])(?=.*\\d)(?=.*[!@#$%^&+=]).{8,}$")) {
                JOptionPane.showMessageDialog(null, "Invalid Password");
                continue;
            }
            registeredCellNumber = prompt("Enter the cell number (+27xxxxxxxxx):");
            if (!registeredCellNumber.matches(cellNumber)) {
                JOptionPane.showMessageDialog(null, "Invalid cell phone number:");
                continue;
            }
            
            JOptionPane.showMessageDialog(null, "Registration succesful!");
            valid = true;
        }
    }
    
    //Checks if the login details match the registered details.
    public static boolean loginUser() {
        String user = prompt("Login. Enter your Username:");
        String password = prompt("Enter your password:");

        return user.equals(registeredUsername) && password.equals(registeredPassword);
    }
    
    //Displays the main menu.
    public static void runMessageSystem() {
        boolean keepRunning = true;
        while (keepRunning) {
            //Added "Stored Messages" option to the main menu
            String[] options = {"Send Messages",  "Stored Messages", "Coming Soon", "Quit"};
            int choice = JOptionPane.showOptionDialog(null, "Choose an option:", "Main Menu",
                    JOptionPane.DEFAULT_OPTION, JOptionPane.INFORMATION_MESSAGE, null, options, options[0]);

            switch (choice) {
                case 0:
                    sendMessages();
                    break;
                case 1:
                    //Added option for "Stored Messages"
                    storedMessagesMenu();
                    break;
                case 2: 
                    JOptionPane.showMessageDialog(null, "Coming Soon.");
                    break;
                default:
                    keepRunning = false;
                    JOptionPane.showMessageDialog(null, "Goodbye!");
                    break;
            }
        }
    }
    
    //Checks how many messages are there and loops.
    public static void sendMessages() {
        int count = 0;
        boolean validCount = false;
        while (!validCount) {
            try {
                count = Integer.parseInt(prompt("Enter the number of messages you would like to send:").trim());
                if (count > 0) {
                    validCount = true;
                } else {
                    JOptionPane.showMessageDialog(null, "Enter a number (must be greater than 0)");
                }
            } catch (NumberFormatException e) {  //Checks for whole numbers.
                JOptionPane.showMessageDialog(null, "Enter a whole number.");
            }
        }

        for (int i = 0; i < count; i++) {
            String messageID = checkMessageID();

            String recipient = prompt("Enter recipient number (+27xxxxxxxxx):").trim();
            String cellCheck = checkRecipientCell(recipient);
            if (!cellCheck.equals("Cell phone number successfully captured.")) {
                JOptionPane.showMessageDialog(null, cellCheck);
                i--;
                continue;
            }

            //Ensures that message block is not blank.
            String message = prompt("Please enter a message of less than 250 characters:");
            if (message.trim().isEmpty()) {
                JOptionPane.showMessageDialog(null, "Message cannot be empty.");
                i--;
                continue;
            }
            
            //Checks for the message length.
            String lengthCheck = checkMessageLength(message);
            if (!lengthCheck.equals("Message ready to send.")) {
                JOptionPane.showMessageDialog(null, lengthCheck);
                i--;
                continue;
            }

            int nextNumber = messageNumber + 1;
            String hash = createMessageHash(messageID, nextNumber, message);
            //Records the IDs and hashes of every message in their arrays
            messageIDs.add(messageID);
            messageHashes.add(hash);

            //Provides the user with options to choose from.
            String[] actions = {"Send Message", "Disregard Message", "Store Message"};
            int action = JOptionPane.showOptionDialog(null, "Choose an option.", "Message Options",
                    JOptionPane.DEFAULT_OPTION, JOptionPane.INFORMATION_MESSAGE, null, actions, actions[0]);

            switch (action) {
                case 0:  //Send
                    messageNumber = nextNumber;
                    displayMessageDetails(messageID, hash, recipient, message);
                    //The array for sent messages contains the message text only.
                    sentMessages.add(message);
                    JOptionPane.showMessageDialog(null, sentMessage(0));
                    break;
                case 2:  //Store
                    messageNumber = nextNumber;
                    storeMessage(messageID, hash, recipient, message);
                    JOptionPane.showMessageDialog(null, sentMessage(2));
                    break;
                default: //Disregard
                    //Stores the disregarded messages in an array.
                    disregardedMessages.add(message);
                    JOptionPane.showMessageDialog(null, sentMessage(1));
                    break;
            }
        }

        JOptionPane.showMessageDialog(null, "Total messages sent: " + returnTotalMessages());
    }
    
    //create a unique 10-digit ID for each message
    public static String checkMessageID() {
        Random rand = new Random();
        String id;
        boolean taken;
        do {
            //Ensures that the digits are randomized.
            id = String.format("%010d", rand.nextInt(1_000_000_000));
            //Checks the message ID array
            taken = messageIDs.contains(id) || storedMessageIDs.contains(id); 
        } while (taken);
        return id;
    }

    public static String createMessageHash(String id, int num, String msg) {
        String[] words = msg.trim().split("\\s+");
        String first = words[0].replaceAll("[^A-Za-z0-9]", "");
        String last = words[words.length - 1].replaceAll("[^A-Za-z0-9]", "");
        return (id.substring(0, 2) + ":" + num + ":" + first + last).toUpperCase();
    }
    
    //Checks the recipeints cellphone number to ensure it matches the regex mentioned above.
    public static String checkRecipientCell(String cell) {
        if (cell.matches(cellNumber)) {
            return "Cell phone number successfully captured.";
        }
        return "Cell phone number is incorrectly formatted or does not contain an international code. Please correct the number and try again.";
    }

    //Checks the message to ensure it does not exceed 250 characters.
    public static String checkMessageLength(String message) {
        if (message.length() <= 250) {
            return "Message ready to send.";
        }
        return "Message exceeds 250 characters by " + (message.length() - 250)
                + "; please reduce the size.";
    }
    
    //0 = send, 1 = disregard, 2 = store
    public static String sentMessage(int action) {
        switch (action) {
            case 0:
                totalMessagesSent++;
                return "Message successfully sent.";
            case 1:
                return "Press 0 to delete the message.";
            case 2:
                return "Message successfully stored.";
            default:
                return "Message disregarded.";
        }
    }

    //Checks how many messages have been sent while program was running.
    public static int returnTotalMessages() {
        return totalMessagesSent;
    }

    public static String printMessages() {
        if (sentMessages.isEmpty()) {
            return "No messages have been sent yet.";
        }
        StringBuilder all = new StringBuilder();
        for (String m : sentMessages) {
            all.append(m).append("\n\n");
        }
        return all.toString();
    }
    
    //Displays the details of the message.
    public static void displayMessageDetails(String messageID, String hash, String recipient, String message) {
        JOptionPane.showMessageDialog(null,
                "Message ID: " + messageID
                + "\nHash: " + hash
                + "\nRecipient: " + recipient
                + "\nMessage: " + message);
    }
    
    //File is now read first so messages stored in earlier runs are kept
    //Manually formats message data as JSON, avoiding external libraries
    //The new message is added to the stored arrays and while file is rewritten from the arrays.
    public static void storeMessage(String messageID, String hash, String recipient, String message) {
        loadStoredMessages();
        storedMessageIDs.add(messageID);
        storedMessageHashes.add(hash);
        storedRecipients.add(recipient);
        storedMessagesFile.add(message);
        saveStoredMessages();
    }
    
    //Turns the stored arrays into one JSON array and wrotes it to jsonnew.json.
    public static void saveStoredMessages() {
        StringBuilder json = new StringBuilder("[\n");
        for (int i = 0; i < storedMessageIDs.size(); i++) {
            json.append("  {\n")
                    .append("    \"MessageID\": \"").append(escapeJson(storedMessageIDs.get(i))).append("\",\n")
                    .append("    \"MessageHash\": \"").append(escapeJson(storedMessageHashes.get(i))).append("\",\n")
                    .append("    \"Recipient\": \"").append(escapeJson(storedRecipients.get(i))).append("\",\n")
                    .append("    \"Message\": \"").append(escapeJson(storedMessagesFile.get(i))).append("\"\n")
                    .append("  }");
            // A comma goes after every entry except the last one.
            if (i < storedMessageIDs.size() - 1) {
                json.append(",");
            }
            json.append("\n");
        }
        json.append("]\n");

        try (FileWriter file = new FileWriter("Jsonnew.json", false)) {
            file.write(json.toString());
        } catch (IOException e) {
            JOptionPane.showMessageDialog(null, "Error writing file: " + e.getMessage());
        }
    }
    
    //Reads the Jsonnew.json and fills storedMessagesFile with each stored message.
    public static void loadStoredMessages() {
        //Clear first so that messages are not duplicated.
        storedMessagesFile.clear();
        storedMessageIDs.clear();
        storedMessageHashes.clear();
        storedRecipients.clear();
        
        Path path = Paths.get("Jsonnew.json");
        if (!Files.exists(path)) {
            return;
        }
        
        try {
            //reads the whole file into one string.
            String json = new String(Files.readAllBytes(Paths.get("Jsonnew.json")));
            //Finds every "Message": "..." value allowing escaped characters inside it.
            storedMessageIDs.addAll(readJsonField(json, "MessageID"));
            storedMessageHashes.addAll(readJsonField(json, "MessageHash"));
            storedRecipients.addAll(readJsonField(json, "Recipient"));
            storedMessagesFile.addAll(readJsonField(json, "Message"));
        } catch (IOException e) {
            JOptionPane.showMessageDialog(null, "Error reading file: " + e.getMessage());
        }
    }
    
    //Finds every value of one key (e.g Recipient) in the JSON text in file order.
    public static ArrayList<String> readJsonField(String json, String key) {
        ArrayList<String> values = new ArrayList<>();
        Matcher matcher = Pattern.compile("\"" + key + "\"\\s*:\\s*\"((?:\\\\.|[^\"\\\\])*)\"").matcher(json);
        while (matcher.find()) {
            values.add(unescapeJson(matcher.group(1)));
        }
        return values;
    }
    
    //Escapes special characters so that the text is safe inside the JSON string.
    public static String escapeJson(String text) {
        return text.replace("\\", "\\\\").replace("\"", "\\\"")
                .replace("\n", "\\n").replace("\r", "\\r").replace("\t", "\\t");
    }
    
    //Reverses the escaping done in storeMessage so the original message text is restored.
    public static String unescapeJson(String text) {
        StringBuilder result = new StringBuilder();
        for (int i = 0; i < text.length(); i++) {
            char c = text.charAt(i);
            if (c == '\\' && i + 1 < text.length()) {
                char next = text.charAt(++i);
                switch (next) {
                    case 'n':
                        result.append('\n');
                        break;
                    case 'r':
                        result.append('\r');
                        break;
                    case 't':
                        result.append('\t');
                        break;
                    default:
                        result.append(next);
                        break;
                }
            } else {
                result.append(c);
            }
        }
        return result.toString();
    }
    
    //Stored messages menu. It reads the file again everytime it opens and then keeps showing the choices until user picks Back or closes the box.
    public static void storedMessagesMenu() {
        loadStoredMessages();
        String[] options = {
            "Sender and recipient of all messages",
            "Longest message",
            "Search by message ID",
            "Search by recipient",
            "Delete by message hash",
            "Full report",
            "Back"
        };
        boolean back = false;

        while (!back) {
            Object choice = JOptionPane.showInputDialog(null, "Choose an option:", "Stored Messages",
                    JOptionPane.QUESTION_MESSAGE, null, options, options[0]);

            // Cancel or Back returns to the main menu.
            if (choice == null || choice.equals("Back")) {
                back = true;
                continue;
            }

            switch ((String) choice) {
                case "Sender and recipient of all messages":
                    showText(displaySendersAndRecipients());
                    break;
                case "Longest message":
                    showText(longestStoredMessage());
                    break;
                case "Search by message ID": {
                    String id = askOptional("Enter the message ID:");
                    if (id != null) {
                        showText(searchByMessageID(id.trim()));
                    }
                    break;
                }
                case "Search by recipient": {
                    String recipient = askOptional("Enter the recipient number (+27xxxxxxxxx):");
                    if (recipient != null) {
                        showText(searchByRecipient(recipient.trim()));
                    }
                    break;
                }
                case "Delete by message hash": {
                    String hash = askOptional("Enter the message hash:");
                    if (hash != null) {
                        showText(deleteByHash(hash.trim()));
                    }
                    break;
                }
                case "Full report":
                    showText(storedMessagesReport());
                    break;
            }
        }
    }
    
    //Lists the sender and recipient of every stored message.
    public static String displaySendersAndRecipients() {
        if (storedMessageIDs.isEmpty()) {
            return "No stored messages.";
        }
        StringBuilder result = new StringBuilder();
        for (int i = 0; i < storedMessageIDs.size(); i++) {
            result.append("Sender: ").append(registeredUsername)
                    .append("\nRecipient: ").append(storedRecipients.get(i))
                    .append("\n\n");
        }
        return result.toString();
    }

    //Finds the stored message with the most characters.
    public static String longestStoredMessage() {
        if (storedMessagesFile.isEmpty()) {
            return "No stored messages.";
        }
        int longestIndex = 0;
        for (int i = 1; i < storedMessagesFile.size(); i++) {
            //Only replaces the longest so far if this one is strictly longer.
            if (storedMessagesFile.get(i).length() > storedMessagesFile.get(longestIndex).length()) {
                longestIndex = i;
            }
        }
        return "Longest stored message:\n" + storedMessagesFile.get(longestIndex);
    }

    //Searches for a message ID and shows its recipient and message.
    public static String searchByMessageID(String id) {
        int index = storedMessageIDs.indexOf(id);
        if (index == -1) {
            return "Message ID not found.";
        }
        return "Recipient: " + storedRecipients.get(index)
                + "\nMessage: " + storedMessagesFile.get(index);
    }

    //Collects every stored message sent to one recipient.
    public static String searchByRecipient(String recipient) {
        StringBuilder result = new StringBuilder();
        for (int i = 0; i < storedRecipients.size(); i++) {
            if (storedRecipients.get(i).equals(recipient)) {
                result.append(storedMessagesFile.get(i)).append("\n\n");
            }
        }
        if (result.length() == 0) {
            return "No stored messages found for " + recipient + ".";
        }
        return "Messages for " + recipient + ":\n\n" + result;
    }

    //Deletes a stored message using its hash then updates the JSON file.
    public static String deleteByHash(String hash) {
        int index = -1;
        for (int i = 0; i < storedMessageHashes.size(); i++) {
            if (storedMessageHashes.get(i).equalsIgnoreCase(hash)) {
                index = i;
                break;
            }
        }
        if (index == -1) {
            return "Message hash not found.";
        }
        String deleted = storedMessagesFile.get(index);
        //Removes the same position from all four arrays so they stay lined up.
        storedMessageIDs.remove(index);
        storedMessageHashes.remove(index);
        storedRecipients.remove(index);
        storedMessagesFile.remove(index);
        saveStoredMessages();
        return "Message \"" + deleted + "\" successfully deleted.";
    }

    //Builds a report with the full details of every stored message.
    public static String storedMessagesReport() {
        if (storedMessageIDs.isEmpty()) {
            return "No stored messages.";
        }
        StringBuilder report = new StringBuilder("STORED MESSAGES REPORT\n\n");
        for (int i = 0; i < storedMessageIDs.size(); i++) {
            report.append("Message ").append(i + 1)
                    .append("\nMessage ID: ").append(storedMessageIDs.get(i))
                    .append("\nMessage Hash: ").append(storedMessageHashes.get(i))
                    .append("\nSender: ").append(registeredUsername)
                    .append("\nRecipient: ").append(storedRecipients.get(i))
                    .append("\nMessage: ").append(storedMessagesFile.get(i))
                    .append("\n\n");
        }
        return report.toString();
    }

    //Returns null on "Cancel" so the user goes back to the menu instead of the whole app closing.
    public static String askOptional(String message) {
        return JOptionPane.showInputDialog(message);
    }

    //Shows text in a scrollable box so long results and reports still fit on screen.
    public static void showText(String text) {
        JTextArea area = new JTextArea(text);
        area.setEditable(false);
        area.setLineWrap(true);
        area.setWrapStyleWord(true);
        JScrollPane scroll = new JScrollPane(area);
        scroll.setPreferredSize(new Dimension(450, 250));
        JOptionPane.showMessageDialog(null, scroll, "Stored Messages", JOptionPane.INFORMATION_MESSAGE);
    }
}

TEST UNITS
package com.mycompany.prog5121w_finalpoe;

import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.util.ArrayList;
import java.util.Arrays;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

public class PROG5121w_FinalPOETest {

    // Test data from the requirements (messages 1-4)
    private static final String MSG1 = "Did you get the cake?";
    private static final String MSG2 = "Where are you? You are late! I have asked you to be on time.";
    private static final String MSG3 = "Ok, I am leaving without you.";
    private static final String MSG4 = "It is dinner time!";

    private static final Path JSON_PATH = Paths.get("Jsonnew.json");
    private byte[] jsonBackup; // holds the real Jsonnew.json so tests never destroy it

    // Resets all static state so tests do not depend on run order
    @BeforeEach
    public void setUp() throws IOException {
        PROG5121w_FinalPOE.sentMessages.clear();
        PROG5121w_FinalPOE.disregardedMessages.clear();
        PROG5121w_FinalPOE.storedMessagesFile.clear();
        PROG5121w_FinalPOE.messageHashes.clear();
        PROG5121w_FinalPOE.messageIDs.clear();
        PROG5121w_FinalPOE.storedMessageIDs.clear();
        PROG5121w_FinalPOE.storedMessageHashes.clear();
        PROG5121w_FinalPOE.storedRecipients.clear();
        PROG5121w_FinalPOE.totalMessagesSent = 0;
        PROG5121w_FinalPOE.messageNumber = 0;
        PROG5121w_FinalPOE.registeredUsername = "test_";

        // Back up the real JSON file, then start each test with no file
        jsonBackup = Files.exists(JSON_PATH) ? Files.readAllBytes(JSON_PATH) : null;
        Files.deleteIfExists(JSON_PATH);
    }

    // Restores the real JSON file (or removes the one the test created)
    @AfterEach
    public void tearDown() throws IOException {
        if (jsonBackup != null) {
            Files.write(JSON_PATH, jsonBackup);
        } else {
            Files.deleteIfExists(JSON_PATH);
        }
    }

    // Sent Messages array correctly populated
    @Test
    public void testSentMessagesArrayPopulated() {
        PROG5121w_FinalPOE.sentMessages.add(MSG1);
        PROG5121w_FinalPOE.sentMessages.add(MSG4);

        ArrayList<String> expected = new ArrayList<>(Arrays.asList(MSG1, MSG4));
        assertEquals(expected, PROG5121w_FinalPOE.sentMessages);
    }

    // Display the longest message
    @Test
    public void testLongestStoredMessage() {
        PROG5121w_FinalPOE.storedMessagesFile.add(MSG1);
        PROG5121w_FinalPOE.storedMessagesFile.add(MSG2);
        PROG5121w_FinalPOE.storedMessagesFile.add(MSG3);
        PROG5121w_FinalPOE.storedMessagesFile.add(MSG4);

        assertEquals("Longest stored message:\n" + MSG2,
                PROG5121w_FinalPOE.longestStoredMessage());
    }

    // Search for message ID (message 4)
    @Test
    public void testSearchByMessageID() {
        PROG5121w_FinalPOE.storedMessageIDs.add("0000000004");
        PROG5121w_FinalPOE.storedRecipients.add("0838884567");
        PROG5121w_FinalPOE.storedMessagesFile.add(MSG4);

        assertEquals("Recipient: 0838884567\nMessage: " + MSG4,
                PROG5121w_FinalPOE.searchByMessageID("0000000004"));
    }

    // Search all messages for one recipient
    @Test
    public void testSearchByRecipient() {
        String recipient = "+27838884567";
        PROG5121w_FinalPOE.storedRecipients.add(recipient);
        PROG5121w_FinalPOE.storedMessagesFile.add(MSG2);
        PROG5121w_FinalPOE.storedRecipients.add(recipient);
        PROG5121w_FinalPOE.storedMessagesFile.add(MSG3);

        assertEquals("Messages for " + recipient + ":\n\n" + MSG2 + "\n\n" + MSG3 + "\n\n",
                PROG5121w_FinalPOE.searchByRecipient(recipient));
    }

    // Delete a message using its hash (test message 2)
    @Test
    public void testDeleteByHash() {
        String hash = PROG5121w_FinalPOE.createMessageHash("0012345678", 2, MSG2); // 00:2:WHERETIME
        PROG5121w_FinalPOE.storedMessageIDs.add("0012345678");
        PROG5121w_FinalPOE.storedMessageHashes.add(hash);
        PROG5121w_FinalPOE.storedRecipients.add("+27838884567");
        PROG5121w_FinalPOE.storedMessagesFile.add(MSG2);

        assertEquals("Message \"" + MSG2 + "\" successfully deleted.",
                PROG5121w_FinalPOE.deleteByHash(hash));
        // All four arrays must stay lined up after the delete
        assertEquals(0, PROG5121w_FinalPOE.storedMessageHashes.size());
        assertEquals(0, PROG5121w_FinalPOE.storedMessagesFile.size());
    }

    // Display report
    @Test
    public void testStoredMessagesReport() {
        PROG5121w_FinalPOE.storedMessageIDs.add("1234567890");
        PROG5121w_FinalPOE.storedMessageHashes.add("12:1:DIDCAKE");
        PROG5121w_FinalPOE.storedRecipients.add("+27834557896");
        PROG5121w_FinalPOE.storedMessagesFile.add(MSG1);

        String expected = "STORED MESSAGES REPORT\n\n"
                + "Message 1\n"
                + "Message ID: 1234567890\n"
                + "Message Hash: 12:1:DIDCAKE\n"
                + "Sender: test_\n"
                + "Recipient: +27834557896\n"
                + "Message: " + MSG1 + "\n\n";
        assertEquals(expected, PROG5121w_FinalPOE.storedMessagesReport());
    }

    //Tests to validate the message
    @Test
    public void testCreateMessageHash() {
        assertEquals("12:1:DIDCAKE",
                PROG5121w_FinalPOE.createMessageHash("1234567890", 1, MSG1));
    }

    @Test
    public void testCheckMessageID() {
        String id = PROG5121w_FinalPOE.checkMessageID();
        assertTrue(id.matches("\\d{10}")); // always exactly 10 digits
    }

    @Test
    public void testCheckRecipientCell() {
        assertEquals("Cell phone number successfully captured.",
                PROG5121w_FinalPOE.checkRecipientCell("+27834557896"));
        assertTrue(PROG5121w_FinalPOE.checkRecipientCell("0838884567")
                .contains("incorrectly formatted"));
    }

    @Test
    public void testCheckMessageLength() {
        assertEquals("Message ready to send.", PROG5121w_FinalPOE.checkMessageLength(MSG1));
        // Boundary is exactly 250 which is allowed, 251 is not
        assertEquals("Message ready to send.",
                PROG5121w_FinalPOE.checkMessageLength("a".repeat(250)));
        assertEquals("Message exceeds 250 characters by 1; please reduce the size.",
                PROG5121w_FinalPOE.checkMessageLength("a".repeat(251)));
    }

    @Test
    public void testSentMessage() {
        assertEquals("Message successfully sent.", PROG5121w_FinalPOE.sentMessage(0));
        assertEquals("Press 0 to delete the message.", PROG5121w_FinalPOE.sentMessage(1));
        assertEquals("Message successfully stored.", PROG5121w_FinalPOE.sentMessage(2));
        assertEquals("Message disregarded.", PROG5121w_FinalPOE.sentMessage(9));
    }

    @Test
    public void testReturnTotalMessages() {
        assertEquals(0, PROG5121w_FinalPOE.returnTotalMessages());
        PROG5121w_FinalPOE.sentMessage(0); // only "send" increases the total
        PROG5121w_FinalPOE.sentMessage(0);
        PROG5121w_FinalPOE.sentMessage(2);
        assertEquals(2, PROG5121w_FinalPOE.returnTotalMessages());
    }

    @Test
    public void testPrintMessages() {
        assertEquals("No messages have been sent yet.", PROG5121w_FinalPOE.printMessages());
        PROG5121w_FinalPOE.sentMessages.add(MSG1);
        PROG5121w_FinalPOE.sentMessages.add(MSG4);
        assertEquals(MSG1 + "\n\n" + MSG4 + "\n\n", PROG5121w_FinalPOE.printMessages());
    }

    //JSON tests

    @Test
    public void testEscapeJson() {
        assertEquals("say \\\"hi\\\"\\nbye", PROG5121w_FinalPOE.escapeJson("say \"hi\"\nbye"));
    }

    @Test
    public void testUnescapeJson() {
        assertEquals("say \"hi\"\nbye", PROG5121w_FinalPOE.unescapeJson("say \\\"hi\\\"\\nbye"));
    }

    @Test
    public void testReadJsonField() {
        String json = "[{\"Recipient\": \"+27834557896\"}, {\"Recipient\": \"+27838884567\"}]";
        ArrayList<String> expected = new ArrayList<>(Arrays.asList("+27834557896", "+27838884567"));
        assertEquals(expected, PROG5121w_FinalPOE.readJsonField(json, "Recipient"));
    }

    // Stores a message, clears the memory and then loads it back from the file
    @Test
    public void testStoreAndLoadStoredMessages() {
        PROG5121w_FinalPOE.storeMessage("1234567890", "12:1:DIDCAKE", "+27834557896", MSG1);

        PROG5121w_FinalPOE.storedMessageIDs.clear();
        PROG5121w_FinalPOE.storedMessageHashes.clear();
        PROG5121w_FinalPOE.storedRecipients.clear();
        PROG5121w_FinalPOE.storedMessagesFile.clear();
        PROG5121w_FinalPOE.loadStoredMessages();

        assertEquals(Arrays.asList("1234567890"), PROG5121w_FinalPOE.storedMessageIDs);
        assertEquals(Arrays.asList("12:1:DIDCAKE"), PROG5121w_FinalPOE.storedMessageHashes);
        assertEquals(Arrays.asList("+27834557896"), PROG5121w_FinalPOE.storedRecipients);
        assertEquals(Arrays.asList(MSG1), PROG5121w_FinalPOE.storedMessagesFile);
    }

    @Test
    public void testLoadStoredMessagesNoFile() {
        PROG5121w_FinalPOE.loadStoredMessages(); // setUp already removed the file
        assertTrue(PROG5121w_FinalPOE.storedMessagesFile.isEmpty());
    }

    //Stored messages menu helpers
    @Test
    public void testDisplaySendersAndRecipients() {
        assertEquals("No stored messages.", PROG5121w_FinalPOE.displaySendersAndRecipients());
        PROG5121w_FinalPOE.storedMessageIDs.add("1234567890");
        PROG5121w_FinalPOE.storedRecipients.add("+27834557896");
        assertEquals("Sender: test_\nRecipient: +27834557896\n\n",
                PROG5121w_FinalPOE.displaySendersAndRecipients());
    }

    @Test
    public void testNotFoundAndEmptyResults() {
        assertEquals("No stored messages.", PROG5121w_FinalPOE.longestStoredMessage());
        assertEquals("Message ID not found.", PROG5121w_FinalPOE.searchByMessageID("999"));
        assertEquals("No stored messages found for +27830000000.",
                PROG5121w_FinalPOE.searchByRecipient("+27830000000"));
        assertEquals("Message hash not found.", PROG5121w_FinalPOE.deleteByHash("XX:0:NONE"));
    }
}
  

