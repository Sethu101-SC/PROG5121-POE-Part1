#PROG5121_POE_Part2
package com.mycompany.prog5121_poe_part2;

import javax.swing.JOptionPane;
import java.util.Random;
import java.io.FileWriter;
import java.io.IOException;
import java.util.ArrayList;

public class PROG5121_POE_Part2 {
    //Declarations for captured registration details which are checked when logging in.
    static String registeredUsername;
    static String registeredPassword;
    static String registeredCellNumber;
    static int totalMessagesSent = 0;   //The counter for messages that user will send is set to 0.
    static int messageNumber = 0; //The counter for the numbering sequence starts at 0.
    static final String cellNumber = "^\\+27[6-8]\\d{8}$"; //The regex that verifies that the cellphone number matches the requirements.
    static ArrayList<String> sentMessages = new ArrayList<>();  //Stores an array of the details of all messages.
    
    //Welcome message. 
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
            String[] options = {"Send Messages", "Coming Soon", "Quit"};
            int choice = JOptionPane.showOptionDialog(null, "Choose an option:", "Main Menu",
                    JOptionPane.DEFAULT_OPTION, JOptionPane.INFORMATION_MESSAGE, null, options, options[0]);

            switch (choice) {
                case 0:  //Send message
                    sendMessages();
                    break;
                case 1:   //Coming soon
                    JOptionPane.showMessageDialog(null, "Coming Soon.");
                    break;
                default:   //Quit
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
            } catch (NumberFormatException e) {     //Checks for a whole number
                JOptionPane.showMessageDialog(null, "Enter a whole number.");
            }
        }

        for (int i = 0; i < count; i++) {
            String messageID = checkMessageID();

            String recipient = prompt("Enter recipient number (+27xxxxxxxxx):");
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

            messageNumber++;
            String hash = createMessageHash(messageID, messageNumber, message);
            
            //Provides the user with options to choose from.
            String[] actions = {"Send Message", "Disregard Message", "Store Message"};
            int action = JOptionPane.showOptionDialog(null, "Choose an option.", "Message Options",
                    JOptionPane.DEFAULT_OPTION, JOptionPane.INFORMATION_MESSAGE, null, actions, actions[0]);

            switch (action) {
                case 0:         //Send: Display the details of the message.
                    displayMessageDetails(messageID, hash, recipient, message);
                    sentMessages.add("Message ID: " + messageID + "\nHash: " + hash
                            + "\nRecipient: " + recipient + "\nMessage: " + message);
                    JOptionPane.showMessageDialog(null, sentMessage(0));
                    break;
                case 2:         //Store: Stores the message to the JSON
                    storeMessage(messageID, hash, recipient, message);
                    JOptionPane.showMessageDialog(null, sentMessage(2));
                    break;
                default:        //Disregard, or dialog closed
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
            taken = false;
            for (String m : sentMessages) {
                if (m.contains("Message ID: " + id)) {
                    taken = true;
                }
            }
        } while (taken);
        return id;
    }

    //Hash format: first two digits of ID : message number : FIRSTWORDLASTWORD
    public static String createMessageHash(String id, int num, String msg) {
        String[] words = msg.trim().split(" ");
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

    //manually formats message data as JSON, avoiding external libraries
    public static void storeMessage(String messageID, String hash, String recipient, String message) {
        String safeMessage = message.replace("\\", "\\\\").replace("\"", "\\\"");

        StringBuilder jsonEntry = new StringBuilder();
        jsonEntry.append("{\n")
                .append("  \"MessageID\": \"").append(messageID).append("\",\n")
                .append("  \"MessageHash\": \"").append(hash).append("\",\n")
                .append("  \"Recipient\": \"").append(recipient).append("\",\n")
                .append("  \"Message\": \"").append(safeMessage).append("\"\n")
                .append("}\n");

        try (FileWriter file = new FileWriter("Jsonnew.json", true)) {
            file.write(jsonEntry.toString());
        } catch (IOException e) {
            JOptionPane.showMessageDialog(null, "Error writing file: " + e.getMessage());
        }
    }
}

TEST UNITS
/*
 * Click nbfs://nbhost/SystemFileSystem/Templates/Licenses/license-default.txt to change this license
 * Click nbfs://nbhost/SystemFileSystem/Templates/UnitTests/JUnit5TestClass.java to edit this template
 */
package PROG5121_POEPART2_UnitTests;


import com.mycompany.prog5121_poe_part2.PROG5121_POE_Part2;
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

/**
 *
 * @author sethu
 */
public class TestData {
        
        @Test
    public void messageLengthSuccess() {
        String msg = "Hi Mike, can you join us for dinner tonight?";

        assertEquals("Message ready to send.",
                PROG5121_POE_Part2.checkMessageLength(msg));
    }

    @Test
    public void messageLengthFailure() {
        String msg = "a".repeat(260);

        assertEquals(
                "Message exceeds 250 characters by 10; please reduce the size.",
                PROG5121_POE_Part2.checkMessageLength(msg));
    }

    @Test
    public void recipientNumberSuccess() {
        assertEquals("Cell phone number successfully captured.",
                PROG5121_POE_Part2.checkRecipientCell("+27718693002"));
    }

    @Test
    public void recipientNumberFailure() {
        assertEquals(
                "Cell phone number is incorrectly formatted or does not contain an international code. Please correct the number and try again.",
                PROG5121_POE_Part2.checkRecipientCell("08575975889"));
    }

    @Test
    public void messageHashCorrect() {
        String[] ids = {"0012345678", "1098765432"};
        int[] nums = {0, 1};

        String[] msgs = {
            "Hi Mike, can you join us for dinner tonight?",
            "Hi Keegan, did you receive the payment?"
        };

        String[] expected = {
            "00:0:HITONIGHT",
            "10:1:HIPAYMENT"
        };

        for (int i = 0; i < msgs.length; i++) {
            assertEquals(expected[i],
                    PROG5121_POE_Part2.createMessageHash(
                            ids[i], nums[i], msgs[i]));
        }
    }

    @Test
    public void messageIdCreated() {
        String id = PROG5121_POE_Part2.checkMessageID();

        assertEquals(10, id.length());
        assertTrue(id.matches("\\d{10}"));
    }

    @Test
    public void messageSent() {
        assertEquals("Message successfully sent.",
                PROG5121_POE_Part2.sentMessage(0));
    }

    @Test
    public void messageDisregarded() {
        assertEquals("Press 0 to delete the message.",
                PROG5121_POE_Part2.sentMessage(1));
    }

    @Test
    public void messageStored() {
        assertEquals("Message successfully stored.",
                PROG5121_POE_Part2.sentMessage(2));
    }

    @Test
    public void totalMessagesSent() {
        int before = PROG5121_POE_Part2.returnTotalMessages();

        PROG5121_POE_Part2.sentMessage(0);
        PROG5121_POE_Part2.sentMessage(1);

        assertEquals(before + 1,
                PROG5121_POE_Part2.returnTotalMessages());
    }
}
    
    



