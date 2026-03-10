# QR Printing In MacOS

Drivers - https://kaywa.me/ipx1s

Currently, the installer package for MacOS can be found here:
*/TSC/MAC Driver/tsc-11.29-install*

<img src="./images/pkg-file-for-macos.png" alt="" width="50%"/>

Follow the installation steps in the wizard. 

>The printer only works via a wire.

>If you face a security warning of the OS, go to *Settings -> Privacy & Security* and grant the necessary permission.

<img src="./images/grant-permission-macos.png" alt="" width="50%"/>

## Add Printer

1. In the System Preferences click the *Printers & Scanners*. Click *Add Printer, Scanner,...* to set up a printer.

<img src="./images/settings-printers-and-scanners-in-macos.png" alt="" width="50%"/>

2. Connect printer to computer through USB. Select the printer model name. Select the *Other*... in the *Use* dropdown menu.

<img src="./images/select-printer-in-macos.png" alt="" width="50%"/>

3. Select the printer PPD file, and click *Open* button.

**PPD folder path : /Library/Printers/TSC/PPDs

<img src="./images/select-pdd-file-in-macos.png" alt="" width="50%"/>

4. Click *Add* button to install the driver.

<img src="./images/install-driver-macos.png" alt="" width="50%"/>

5. Done.

<img src="./images/done-macos.png" alt="" width="50%"/>

## Setting up Printing Options
1. In the browser, press *Cmd + P* or choose *File -> Print* in the browser top bar.

<img src="./images/print-in-browser-macos.png" alt="" width="50%"/>

2. Select the TSC TE200 printer, then click *More settings* and *Print using system dialog*.

<img src="./images/print-in-browser-settings-macos.png" alt="" width="50%"/>

3. Make sure the TSC TE200 printer is selected, then click the *Paper Size* dropdown.

<img src="./images/print-in-browser-configuration-macos.png" alt="" width="50%"/>

In the dropdown choose *Manage Custom Sizes*.

<img src="./images/print-in-browser-manage-custom-sizes-macos.png" alt="" width="50%"/>

4. In the window that opens, add a configuration, fill in the configuration name, set the width to 58mm (2.28 inches) and height to 30mm (1.18 inches), and then click OK.

<img src="./images/print-in-browser-set-dimensions-macos.png" alt="" width="50%"/>

## Printing QR Codes

1. After configuring the printing settings, you can start printing. To do this, go to the page of the book for which you need to print the QR code.

<img src="./images/book-page.png" alt="" width="50%"/>

2. Click on the *Print* button.

<img src="./images/modal-qr.png" alt="" width="50%"/>

3. In the print window, select the TSC TE200 printer, select *Landscape* in the Layout field, go to *More Settings -> Print using system dialog* .

<img src="./images/print-in-browser-settings-macos.png" alt="" width="50%"/>

4. Select the TSC TE200 printer and use the configuration you created. Click the *Print* button, after that, the printing process should start.

<img src="./images/print-in-browser-use-configuraion-macos.png" alt="" width="50%"/>
