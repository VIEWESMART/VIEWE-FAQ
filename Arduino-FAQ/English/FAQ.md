# FAQ

* [中文版本](./FAQ_CN.md)

## Table of Contents

- [FAQ](#faq)
  - [Table of Contents](#table-of-contents)
  - [Where is the directory for Arduino libraries?](#where-is-the-directory-for-arduino-libraries)
  - [How to Install ESP32\_Display\_Panel in Arduino IDE?](#how-to-install-esp32_display_panel-in-arduino-ide)
  - [Where are the installation directory for arduino-esp32 and the SDK located?](#where-are-the-installation-directory-for-arduino-esp32-and-the-sdk-located)
  - [How to fix screen drift issue when driving RGB LCD with ESP32-S3?](#how-to-fix-screen-drift-issue-when-driving-rgb-lcd-with-esp32-s3)
  - [How to Use ESP32\_Display\_Panel on PlatformIO?](#how-to-use-esp32_display_panel-on-platformio)
  - [How to add an LVGL library and how to configure?](#How-to-add-an-LVGL-library-and-how-to-configure)

## Where is the directory for Arduino libraries?

You can find and modify the directory path for Arduino libraries by selecting `File` > `Preferences` > `Settings` > `Sketchbook location` from the menu bar in the Arduino IDE.

## How to Install ESP32_Display_Panel in Arduino IDE?

- If you want to install online, navigate to `Sketch` > `Include Library` > `Manage Libraries...` in the Arduino IDE, then search for `ESP32_Display_Panel` and click the `Install` button to install it.
- If you want to install manually, download the required version of the `.zip` file from [ESP32_Display_Panel](https://github.com/esp-arduino-libs/ESP32_Display_Panel), then navigate to `Sketch` > `Include Library` > `Add .ZIP Library...` in the Arduino IDE, select the downloaded `.zip` file, and click `Open` to install it.
- You can also refer to the guides on library installation in the [Arduino IDE v1.x.x](https://docs.arduino.cc/software/ide-v1/tutorials/installing-libraries) or [Arduino IDE v2.x.x](https://docs.arduino.cc/software/ide-v2/tutorials/ide-v2-installing-a-library) documentation.

## Where are the installation directory for arduino-esp32 and the SDK located?

The default installation path for arduino-esp32 depends on your operating system:

- Windows: `C:\Users\<user name>\AppData\Local\Arduino15\packages\esp32`
- Linux: `~/.arduino15/packages/esp32`

The SDK for arduino-esp32 v3.x.x version is located in the `tools/esp32-arduino-libs/idf-release_x` directory within the default installation path.

## How to fix screen drift issue when driving RGB LCD with ESP32-S3?

When encountering screen drift issue when driving RGB LCD with ESP32-S3, you can follow these steps to resolve them:

1. **Refer to Documentation**: Understand the issue description in detail, you can refer to [this document](https://docs.espressif.com/projects/esp-faq/en/latest/software-framework/peripherals/lcd.html#why-do-i-get-drift-overall-drift-of-the-display-when-esp32-s3-is-driving-an-rgb-lcd-screen).

2. **Enable `Bounce Buffer + XIP on PSRAM` Feature**: To resolve the issue, it's recommended to enable the `Bounce Buffer + XIP on PSRAM` feature. Follow these steps:

   - **Step 1**: Download the "high_perf" version of the SDK from [arduino-esp32-sdk](https://github.com/esp-arduino-libs/arduino-esp32-sdk) and replace it in the [installation directory of arduino-esp32](#where-are-the-installation-directory-for-arduino-esp32-and-the-sdk-located).

   - **Step 2**: If you are using supported development boards, usually there's no need to modify the code as they set `ESP_PANEL_LCD_RGB_BOUNCE_BUF_SIZE` to `(ESP_PANEL_LCD_WIDTH * 10)` by default. If the issue persists, refer to the example code below to increase the size of the `Bounce Buffer`.

   - **Step 3**: If you are using a custom board, confirm in the `ESP_Panel_Board_Custom.h` file whether `ESP_PANEL_LCD_RGB_BOUNCE_BUF_SIZE` is set to non-zero. If the issue persists, increase the size of the `Bounce Buffer`.

   - **Step 4**: If you are using an independent driver, refer to the example code below to set the size of the `Bounce Buffer`.

   - **Step 5**: If you are developing an LVGL application, assign the task that initializes the RGB peripheral and the task that runs the LVGL `lv_timer_handler()` on the same core. Please refer to [the code](../examples/LVGL/v8/Porting/lvgl_port_v8.h#L53).

3. **Example Code**: The following example code demonstrates how to modify the size of the `Bounce Buffer` using `ESP_Panel` driver or independent driver:

   **Example 1**: Modify the `Bounce Buffer` size using the `ESP_Panel` driver:

    ```c
    ...
    ESP_Panel *panel = new ESP_Panel();
    panel->init();
    // Start
    ESP_PanelBus_RGB *rgb_bus = static_cast<ESP_PanelBus_RGB *>(panel->getLcd()->getBus());
    // The size of the bounce buffer must satisfy `width_of_lcd * height_of_lcd = size_of_buffer * N`, where N is an even number.
    rgb_bus->configRgbBounceBufferSize((ESP_PANEL_LCD_WIDTH * 20));
    // End
    panel->begin();
    ...
    ```

   **Example 2**: Modify the `Bounce Buffer` size using an independent driver:

    ```c
    ...
    ESP_PanelBus_RGB *lcd_bus = new ESP_PanelBus_RGB(...);
    // Start
    // The size of the bounce buffer must satisfy `width_of_lcd * height_of_lcd = size_of_buffer * N`, where N is an even number.
    lcd_bus->configRgbBounceBufferSize(EXAMPLE_LCD_WIDTH * 10);
    // End
    lcd_bus->begin();
    ...
    ```

## How to Use ESP32_Display_Panel on PlatformIO?

You can refer to the example [PlatformIO](../examples/PlatformIO/) to use the ESP32_Display_Panel library in PlatformIO. By default, it is suitable for the **ESP32-S3-LCD-EV-Board** and **ESP32-S3-LCD-EV-Board-2** development boards. You need to modify the [boards/ESP-LCD.json](../examples/PlatformIO/boards/ESP-LCD.json) file according to the actual situation.

## How to add an LVGL library and how to configure?
* How to use it normally

  LVGL's features and parameters can be adjusted by modifying the [lv_conf.h](../../template_files/lv_conf.h) configuration file. Here's how to configure LVGL:

  1. **Configuration File Search Rules**

  - When using arduino-esp32 v3 version, LVGL searches for configuration files in priority order: `Current project directory` > `Arduino library directory`
  - If no configuration file is found, a warning will be shown during compilation
  - Please ensure at least one directory contains the *lv_conf.h* file

  2. **Configuration File Usage**

  - Single project configuration: Place the configuration file in the project directory
  - Multiple projects sharing configuration: Place the configuration file in the [Arduino library directory](#where-is-the-arduino-library-directory)

  3. **Configuration Steps**
  
      a. Get configuration file template
  
      - Navigate to [Arduino library directory](#where-is-the-arduino-library-directory)
      - Enter the *lvgl* folder
      - Copy *lv_conf_template.h* to target directory and rename it to *lv_conf.h*
  
      b. Enable configuration file
  
      - Open *lv_conf.h*
      - Change the first `#if 0` to `#if 1`
  
      c. Common configuration items
  
        ```c
        // Color configuration
        #define LV_COLOR_DEPTH          16    // Usually use 16-bit color depth (RGB565)
                                                // Set to 32 to support 24-bit color depth with alpha (ARGB8888)
        #define LV_COLOR_16_SWAP        0     // SPI/QSPI LCD (like ESP32-C3-LCDkit) needs to set to 1
        #define LV_COLOR_SCREEN_TRANSP  1     // LCD with alpha needs to set to 1
        // Memory configuration
        #define LV_MEM_CUSTOM           1     // Use malloc/free for better performance
        #define LV_MEMCPY_MEMSET_STD    1     // Use standard library functions
        // Resource configuration
        #define LV_FONT_MONTSERRAT_N    1     // Enable required built-in fonts (replace N with font size)
        // Debug configuration
        #define LV_USE_PERF_MONITOR     1     // Display CPU usage and FPS
        #define LV_USE_LOG              1     // Enable logging feature
        #define LV_LOG_PRINTF           1     // Use printf for log output
        // Other configuration
        #define LV_ATTRIBUTE_FAST_MEM   IRAM_ATTR  // Improve performance but will use more SRAM
        ```
  
* How to use the examples and demos in lvgl

  1.Copy the `demos` folder and `examples` folder in the lvgl library file to the `src` folder
  
  2.please open the corresponding macros
  ```c
  ...
  /*==================
  * EXAMPLES
  *==================*/
  
  /*Enable the examples to be built with the library*/
  #define LV_BUILD_EXAMPLES 1
  
  /*===================
   * DEMO USAGE
   ====================*/
  
  /*Show some widget. It might be required to increase `LV_MEM_SIZE` */
  #define LV_USE_DEMO_WIDGETS 1
  #if LV_USE_DEMO_WIDGETS
  #define LV_DEMO_WIDGETS_SLIDESHOW 0
  #endif
  
  /*Demonstrate the usage of encoder and keyboard*/
  #define LV_USE_DEMO_KEYPAD_AND_ENCODER 0
  
  /*Benchmark your system*/
  #define LV_USE_DEMO_BENCHMARK 0
  #if LV_USE_DEMO_BENCHMARK
  /*Use RGB565A8 images with 16 bit color depth instead of ARGB8565*/
  #define LV_DEMO_BENCHMARK_RGB565A8 0
  #endif
  
  /*Stress test for LVGL*/
  #define LV_USE_DEMO_STRESS 0
  
  /*Music player demo*/
  #define LV_USE_DEMO_MUSIC 0
  #if LV_USE_DEMO_MUSIC
      #define LV_DEMO_MUSIC_SQUARE    0
      #define LV_DEMO_MUSIC_LANDSCAPE 0
      #define LV_DEMO_MUSIC_ROUND     0
      #define LV_DEMO_MUSIC_LARGE     0
      #define LV_DEMO_MUSIC_AUTO_PLAY 0
  #endif

  ...
  ```
