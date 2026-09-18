# PROG5121-POE-Part1
POE Part 1
/*
 * Click nbfs://nbhost/SystemFileSystem/Templates/Licenses/license-default.txt to change this license
 */
package com.mycompany.prog5121_poe_part1;

import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;
import java.security.SecureRandom;
import java.util.ArrayList;  //Import the java utility for arrays
import java.util.Scanner;   //Import the scanner utility
/**
 *
 * @author sethu
 */
public class PROG5121_POE_Part1 {

    public static void main(String[] args) {
        Scanner sethu = new Scanner(System.in);
        ArrayList<Login> users = new ArrayList<>();   //Array list is used to store the registered users

        System.out.println("\n Welcome to our chat app 2026! Select your option below:");

        while (true) {
            //Menu display after each action loop until user chooses option 3
            System.out.println("\n Main Menu:");
            System.out.println("1. Register");
            System.out.println("2. Login");
            System.out.println("3. Exit");
            System.out.println("Choose an option:");

            int choice = sethu.nextInt(); //Declaring the choice number as integer
            sethu.nextLine();

            switch (choice) {
                case 1: {   //Register the user
                    System.out.println("Enter first name:");
                    String firstName = sethu.nextLine();
                    System.out.println("Enter last name:");
                    String lastName = sethu.nextLine();
                    System.out.println("Full name successfully captured!");

                    //USERNAME LOOP
                    //Setting and checking the conditions to make sure username includes underscore and <=5 characters long
                    String username;
                    boolean validUsername;
                    do {
                        System.out.println("Enter username:");
                        username = sethu.nextLine();

                        validUsername = username.contains("_") && username.length() <= 5;

                        if (!validUsername) {
                            System.out.println("Username is not correctly formatted; please ensure that your username contains an underscore and is no more than five characters in length.");
                            continue;  //skips the duplicate check and goes straight to re-prompting
                        }
                        
                        final String checkUsername = username;
                        boolean userExists = users.stream().anyMatch(u -> u.getUsername().equals(checkUsername));
                        if (userExists){
                            System.out.println("Username already exists.Please choose another username");validUsername = false;
                        }  //This ensures the loop runs again
                    
                    } while (!validUsername);
                    System.out.println("Username successfully captured!");

                    //PASSWORD LOOP
                    //Setting the password conditions
                    //At least 8 characters, at least one uppercase letter, at least one digit, and at least one special character.
                    String password;
                    boolean validPassword;
                    do {
                        System.out.println("Enter password:");
                        password = sethu.nextLine();

                        validPassword = password.length() >= 8                                   //at least 8 characters
                                && password.matches(".*[A-Z].*")                                  //contains an uppercase letter
                                && password.matches(".*\\d.*")                                    //contains a digit
                                && password.matches(".*[!@#$%^&*(),.?\\\":{}|<>].*");             //contains a special character

                        if (!validPassword) {
                            System.out.println("Password is not correctly formatted; please ensure that the password contains at least eight characters, a capital letter, a number, and a special character.");
                        }
                    } while (!validPassword);
                    System.out.println("Password successfully captured!");

                    //CELLPHONE NUMBER LOOP
                    //Setting and checking the conditions to make sure the cellphone number has an SA country code (+27)
                    //followed by 6, 7 or 8, (9 digits total after +27).
                    String cellphoneNumber;
                    String regex = "\\+[27][0-9]{10}";
                    do {
                        System.out.println("Enter a South African cellphone number (e.g. +27821111111):");
                        cellphoneNumber = sethu.nextLine();

                        if (!cellphoneNumber.matches(regex)) {
                            System.out.println("Cellphone number incorrectly formatted or does not contain South African code, e.g. +27821111111");
                        }
                    } while (!cellphoneNumber.matches(regex));
                    System.out.println("Cellphone number is successfully added.");

                    //If username already exists, the below detects duplicates.
                    final String finalUsername = username;
                    boolean userExists = users.stream().anyMatch(u -> u.getUsername().equals(finalUsername));
                    if (userExists) {
                        System.out.println("Username already exists. Please choose another username.");
                        validUsername = false;
                    }

                    //Once the above have successfully been checked, the user will be created and stored.
                    //Password hashing/salting happens inside the Login constructor.
                    users.add(new Login(firstName, lastName, username, password, cellphoneNumber));
                    System.out.println("User registered successfully!");

                    //Welcome the user and move to the login.
                    System.out.println("Welcome " + firstName + " " + lastName + "! You can now login.");
                    System.out.println("Redirecting you to the login page...\n");
                    promptLogin(sethu, users);
                    break;
                }

                case 2:  //Login prompt
                    if (users.isEmpty()) {
                        //If there are no credentials to check against yet.
                        System.out.println("Username does not exist. Please register.");
                    } else {
                        promptLogin(sethu, users);
                    }
                    break;

                case 3: // Exit option
                    System.out.println("Goodbye!");
                    sethu.close(); //closes the scanner
                    return;        
                default:
                    //If user enters any option outside 1-3
                    System.out.println("Invalid option. Please select 1, 2 or 3");
            }
        }
    }


    //handling login logic
    private static void promptLogin(Scanner sethu, ArrayList<Login> users) {
        //prompt user to enter username and password
        System.out.println("Enter username:");
        String enteredUsername = sethu.nextLine();
        System.out.println("Enter password:");
        String enteredPassword = sethu.nextLine();

        //search account by username: if no match is found then null
        Login user = users.stream()
                .filter(u -> u.getUsername().equals(enteredUsername))
                .findFirst()
                .orElse(null);

        //Password will be checked only if the user is found
        boolean loginStatus = user != null && user.loginUser(enteredPassword);
        System.out.println(user != null ? user.returnLoginStatus(loginStatus) : "User not found.");
    }
}

//Registered account
class Login {
    public String firstName;
    public String lastName;
    public String username;
    public String passwordHash;  //SHA-256 hash of (salt + password), stored as hex
    public String cellphoneNumber;
    public byte[] salt;  //random per-user salt, generated once at registration

    public Login(String firstName, String lastName, String finalUsername, String password, String cellphoneNumber) {
        this.firstName = firstName;
        this.lastName = lastName;
        this.username = finalUsername;
        this.salt = generateSalt();  // unique salt for the user
        this.passwordHash = hashPassword(password, salt);  //hash + salt password immediately
        this.cellphoneNumber = cellphoneNumber;
    }

    public String getUsername() {
        return username;
    }

    public String getFullName() {
        return firstName + " " + lastName;
    }

    //checks login attempts
    public boolean loginUser(String enteredPassword) {
        return this.passwordHash.equals(hashPassword(enteredPassword, salt));
    }

    //converts login pass/fail into message shown to user.
    public String returnLoginStatus(boolean status) {
        return status ? "Login successful! Welcome back, " + getFullName() + "!" : "Invalid username or password.";
    }

    private String hashPassword(String password, byte[] salt) {
        try {
            MessageDigest md = MessageDigest.getInstance("SHA-256");
            md.update(salt); // mix in the salt before the password bytes
            byte[] hash = md.digest(password.getBytes());

            // Convert the raw hash bytes into a readable hex string
            StringBuilder hexString = new StringBuilder();
            for (byte b : hash) {
                hexString.append(String.format("%02x", b));
            }
            return hexString.toString();
        } catch (NoSuchAlgorithmException e) {
            
            throw new RuntimeException("Error hashing password", e);
        }
    }

    private byte[] generateSalt() {
        SecureRandom random = new SecureRandom();
        byte[] salt = new byte[16];
        random.nextBytes(salt);
        return salt;
    }
}

Unit Test

/*
 * Click nbfs://nbhost/SystemFileSystem/Templates/Licenses/license-default.txt to change this license
 * Click nbfs://nbhost/SystemFileSystem/Templates/UnitTests/JUnit5TestClass.java to edit this template
 */
package com.mycompany.prog5121_poe_part1;

import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

/**
 *
 * @author sethu
 */
public class PROG5121_POE_Part1IT {
    
    public PROG5121_POE_Part1IT() {
    }

    @Test
    public void testMain() {
    }
    
}

