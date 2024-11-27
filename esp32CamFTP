#include "esp_camera.h"
#include <WiFi.h>
#include <ESP32_FTPClient.h>
#include <time.h>

// Replace with your network credentials
const char* ssid = "NET-MESH-FOREST";
const char* password = "B4r3f2c1!+";

// Replace with your FTP server credentials
char ftp_server[] = "192.168.88.250";
char ftp_user[] = "amb82";
char ftp_pass[] = "";
// uint16_t ftp_port = 21; // Default FTP port, change if necessary

// Camera module pin definitions for AI-Thinker
#define PWDN_GPIO_NUM    32
#define RESET_GPIO_NUM   -1
#define XCLK_GPIO_NUM    0
#define SIOD_GPIO_NUM    26
#define SIOC_GPIO_NUM    27

#define Y9_GPIO_NUM      35
#define Y8_GPIO_NUM      34
#define Y7_GPIO_NUM      39
#define Y6_GPIO_NUM      36
#define Y5_GPIO_NUM      21
#define Y4_GPIO_NUM      19
#define Y3_GPIO_NUM      18
#define Y2_GPIO_NUM      5
#define VSYNC_GPIO_NUM   25
#define HREF_GPIO_NUM    23
#define PCLK_GPIO_NUM    22

// Initialize the FTP Client
ESP32_FTPClient ftp(ftp_server, ftp_user, ftp_pass/*, ftp_port*/);

void startCameraServer();

// Camera configuration
void setup() {
  Serial.begin(115200);
  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("WiFi connected");

  // Initialize time
  configTime(0, 0, "pool.ntp.org");  // Get time from NTP server

  // Wait for time to be set
  while (!time(nullptr)) {
    Serial.print("*");
    delay(1000);
  }
  Serial.println("Time set");

  camera_config_t config;
  config.ledc_channel = LEDC_CHANNEL_0;
  config.ledc_timer = LEDC_TIMER_0;
  config.pin_d0 = Y2_GPIO_NUM;
  config.pin_d1 = Y3_GPIO_NUM;
  config.pin_d2 = Y4_GPIO_NUM;
  config.pin_d3 = Y5_GPIO_NUM;
  config.pin_d4 = Y6_GPIO_NUM;
  config.pin_d5 = Y7_GPIO_NUM;
  config.pin_d6 = Y8_GPIO_NUM;
  config.pin_d7 = Y9_GPIO_NUM;
  config.pin_xclk = XCLK_GPIO_NUM;
  config.pin_pclk = PCLK_GPIO_NUM;
  config.pin_vsync = VSYNC_GPIO_NUM;
  config.pin_href = HREF_GPIO_NUM;
  config.pin_sscb_sda = SIOD_GPIO_NUM;
  config.pin_sscb_scl = SIOC_GPIO_NUM;
  config.pin_pwdn = PWDN_GPIO_NUM;
  config.pin_reset = RESET_GPIO_NUM;
  config.xclk_freq_hz = 20000000;
  config.pixel_format = PIXFORMAT_JPEG;

  if(psramFound()){
    config.frame_size = FRAMESIZE_UXGA;
    config.jpeg_quality = 10;
    config.fb_count = 2;
  } else {
    config.frame_size = FRAMESIZE_SVGA;
    config.jpeg_quality = 12;
    config.fb_count = 1;
  }

  // Initialize the camera
  esp_err_t err = esp_camera_init(&config);
  if (err != ESP_OK) {
    Serial.printf("Camera init failed with error 0x%x", err);
    return;
  }

  startCameraServer();

  // Initialize FTP
  ftp.OpenConnection();

  // Take a picture and upload it every minute
  while (true) {
    takeAndUploadPicture();
    delay(60000); // 1 minute delay
  }
}

void takeAndUploadPicture() {
  camera_fb_t * fb = NULL;

  // Take a picture
  fb = esp_camera_fb_get();
  if (!fb) {
    Serial.println("Camera capture failed");
    return;
  }

  // Get current time
  time_t now = time(nullptr);
  struct tm * timeinfo = localtime(&now);

  // Create filename with date and time
  char filename[30];
  strftime(filename, sizeof(filename), "/picture_%Y%m%d_%H%M%S.jpg", timeinfo);

  // FTP upload
  ftp.ChangeWorkDir("/"); // change directory to root
  ftp.InitFile("Type I");
  ftp.NewFile(filename);
  ftp.WriteData((unsigned char*)fb->buf, fb->len);
  ftp.CloseFile();

  Serial.println("Picture uploaded");

  // Return the frame buffer back to the driver for reuse
  esp_camera_fb_return(fb);
}

void startCameraServer() {
  // Code to start camera server, if needed
}

void loop() {
  // Empty loop since the main code is in setup
}
