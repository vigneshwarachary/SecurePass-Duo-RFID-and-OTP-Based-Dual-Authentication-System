# SecurePass-Duo-RFID-and-OTP-Based-Dual-Authentication-System
# LPC2148-RFID-OTP-Smart-Security-Door-System  

This project is a two-factor authentication based security door system built on the LPC2148 microcontroller. It allows access only to authorized users by first   verifying an RFID card and then sending a time-based OTP via GSM SMS to a registered mobile number. The door is controlled automatically based on successful   authentication.  

## Features  

1. Two-Factor Authentication (RFID + OTP)  
2. GSM-Based OTP via SMS  
3. OTP Expiry and Attempt Limit  
4. Admin RTC Edit Mode  
5. Real-Time Clock Display  

## Required Components   

1. LPC2148 Board  
2. RFID Reader Module  
3. GSM Module  
4. LCD (16x2)  
5. Keypad Matrix (4x4)   
## Tools  
Compiler: Keil uVision4  
Flashing Tool: Flash Magic  

## System Architecture and Component Roles  

The system is built around the LPC2148, which acts as the intelligent hub for all peripheral interactions. The architecture is designed for reliability, ensuring that user commands are processed securely and executed predictably.  
 
1. The Central Controller: LPC2148  
   The LPC2148 manages the entire execution flow. It handles:  
   - Peripheral Management: Managing the communication protocols (UART0 for GSM, UART1 for RFID, GPIO for door lock and LEDs) required to interface with external   hardware.  
   - Logic Processing: Running the main control loop for RFID validation, OTP generation, OTP verification, and door control.  
   - Interrupt Handling: Managing EINT1 (External Interrupt) to allow the admin to edit RTC settings without interrupting standard system operation.  

2. Security & Data Management:  
   - RFID Reader: Acts as the primary security gateway. It provides a contactless interface to identify the user before the OTP process begins, preventing   unauthorized access attempts.  
   - GSM Module: Serves as the OTP delivery channel. Upon valid RFID scan, it sends a 4-digit time-seeded OTP to a registered mobile number using AT+CMGS command   in text mode (AT+CMGF=1).  

3. User Interface & Feedback:  
   - Keypad Matrix: Allows the user to enter the received OTP. It supports digit input, backspace (key 15), and confirm (key 16).  
   - LCD Module: Acts as the visual dashboard, providing immediate feedback. It displays system status (e.g., "Valid Tag", "WRONG OTP", "Access Granted"), prompts   for OTP entry, and shows the current RTC time and date.  

## Set-up Instructions  

1. Before using the peripherals you must initialize them by calling InitUART0(), InitUART1(), InitLCD(), RTC_Init(), gsm_init(), InitKPM(), and enable_ent1().  
2. Note that you must include all the required headers like "lpc21xx.h", "string.h" and user-defined headers like "lcd_defines.h", "lcd.h", "delay.h", "types.h",   "uart_interuppt.h", "kpm.h", "rtc.h", "admin.h", "defines.h".  
   - "lcd_defines.h" contains the command values and the pin connections of LCD.  
   - "lcd.h" contains the function declarations of LCD.  
   - "delay.h" contains the function declarations of delay functions according to the time of delay required.  
   - "types.h" contains the type casted details of the existing data types.  
   - "uart_interuppt.h" contains the function declarations of UART0 and UART1 related operations.  
   - "defines.h" contains the macro expansions of Bit/Byte manipulation.  
   - "kpm.h" contains the function declarations of the Keypad related operations.  
   - "rtc.h" contains the function declarations of the RTC related operations.  
   - "admin.h" contains the function declarations of the Admin mode operations.  

## Code Execution Flow  

```
START  
Initialize LCD, Keypad, UART0 (GSM), UART1 (RFID), RTC, GSM Module, External Interrupt (EINT1)  

LOOP (Main System):  
    Display current RTC Time, Date, and Day on LCD  

    WAIT for RFID card scan (UART1 interrupt sets r_flag1 = 2)  

    Read RFID tag from buff1[]  
    IF (tag matches valid tag):  
        Display "Valid Tag"  
        Store current RTC time as otp_time  
        generate_otp()   // seed OTP from hour, min, sec  
        send_otp()       // send OTP via GSM SMS   

        attempts = 0  
        WHILE (attempts < 3):  
            validate_otp()   // prompt user to enter OTP on keypad  
            IF (OTP entered matches generated OTP):  
                check_otp()  // verify within 60 seconds  
                IF (within time limit):  
                    Display "Access Granted"  
                    Open Door (P0.12 HIGH)  
                    Delay  
                    Close Door (P0.12 LOW)  
                    BREAK  
                ELSE:  
                    Display "OTP EXPIRED"  
                    BREAK  
            ELSE:  
                attempts++  
                Display "WRONG OTP"  
                IF (attempts >= 3):  
                    Display "Access Denied"  
    ELSE:  
        Display "Invalid Card"  

END LOOP  

ISR (EINT1 - Admin Mode):  
    Call adminmode()  
    adminmode() prompts:  
        1. RTC Edit (Hour, Minute, Second, Date, Month, Year, Day)  
        2. Exit  
    Clear EXTINT flag  
    RETURN from ISR  

## Pin Connections  

1. Connect P0.0 to the TxD pin of the GSM Module (UART0 Tx).  
2. Connect P0.1 to the RxD pin of the GSM Module (UART0 Rx).  
3. Connect P0.3 to the switch to raise an External Interrupt because P0.3 supports the EINT1 functionality (pin function 2) in LPC2148.  
4. Connect P0.8 to the TxD pin of the RFID Reader (UART1 Tx).  
5. Connect P0.9 to the RxD pin of the RFID Reader (UART1 Rx).  
6. Connect the pins P0.8 to P0.15 to the LCD Data lines (D0–D7).  
7. Connect the pin P0.16 to RS (Register Select) of LCD.  
8. Connect the pin P0.17 to RW (Read/Write) of LCD.  
9. Connect the pin P0.18 to EN (Enable) of LCD.  
10. Connect P0.12 to the Relay/Solenoid (Door Lock control — HIGH = Open, LOW = Close).  
11. Connect the VCC and GND pin of the GSM Module to the appropriate power supply and GND respectively.  
12. Connect the Keypad from the pins P1.16 to P1.23 (Rows: P1.16–P1.19, Columns: P1.20–P1.23).  

## How to Use  

1. System Power-Up:  
   Upon connecting the power supply after loading the code into the hardware, the system will initialize all peripherals and the GSM module. The LCD will display   the current time and date continuously.  

2. RFID Authentication:  
   Bring your registered RFID card near the reader. The system reads the tag via UART1 interrupt. If the tag matches the stored valid tag, the system proceeds to   OTP generation. If the tag is invalid, the LCD displays "Invalid Card" and returns to the main loop.  

3. OTP Verification:  
   After a valid RFID scan, a 4-digit OTP is generated using the current RTC time and sent as an SMS to the registered mobile number via the GSM module. Enter the   received OTP using the Keypad Matrix within 60 seconds. The system allows a maximum of 3 attempts. If the OTP is correct and within the time limit, the door   opens automatically.  

4. Changing RTC Settings (Admin Mode):  
   * If you need to update the RTC time or date, press the physical button connected to the External Interrupt (EINT1) pin P0.3.  
   * The system will enter Admin Mode and display the admin menu on the LCD.  
   * Select option 1 to edit RTC values (Hour, Minute, Second, Date, Month, Year, Day of Week).  
   * Select option 2 to exit Admin Mode and return to the main loop.  
   Note: Always ensure the RFID scan and OTP flow completes before entering Admin Mode to avoid interrupting an active authentication session.  

## Future Improvements  

1. Remote Monitoring & Connectivity: Integrate a Wi-Fi or Bluetooth module to allow caretakers to monitor door access logs or trigger the door remotely via a   smartphone app.  
2. Intelligent OTP Delivery: Improve OTP security by using a stronger algorithm (e.g., TOTP standard) and adding an encrypted delivery channel instead of plain   SMS.  
3. Access Logging with RTC Timestamps: Expand the system to log every RFID scan attempt (valid or invalid) with an RTC timestamp stored to an external EEPROM or   SD card for audit purposes.  
