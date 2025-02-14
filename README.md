# -Translation-
Language Translation Tool
#include <iostream>
#include <curl/curl.h>
#include <string>
#include <json/json.h>  // JSON parsing ke liye

using namespace std;

// API se response store karne ke liye callback function
size_t WriteCallback(void* contents, size_t size, size_t nmemb, string* output) {
    size_t totalSize = size * nmemb;
    output->append((char*)contents, totalSize);
    return totalSize;
}

string translateText(const string& text, const string& sourceLang, const string& targetLang, const string& apiKey) {
    CURL* curl;
    CURLcode res;
    string readBuffer;

    string url = "https://translation.googleapis.com/language/translate/v2?key=" + apiKey +
                 "&q=" + curl_easy_escape(curl, text.c_str(), text.length()) +
                 "&source=" + sourceLang + "&target=" + targetLang;

    curl = curl_easy_init();
    if (curl) {
        curl_easy_setopt(curl, CURLOPT_URL, url.c_str());
        curl_easy_setopt(curl, CURLOPT_WRITEFUNCTION, WriteCallback);
        curl_easy_setopt(curl, CURLOPT_WRITEDATA, &readBuffer);

        res = curl_easy_perform(curl);
        if (res != CURLE_OK) {
            cerr << "cURL request failed: " << curl_easy_strerror(res) << endl;
        }

        curl_easy_cleanup(curl);
    }

    // JSON response parse karna
    Json::Value jsonData;
    Json::CharReaderBuilder reader;
    string errors;
    string translatedText = "Translation failed";

    if (Json::parseFromStream(reader, readBuffer, &jsonData, &errors)) {
        translatedText = jsonData["data"]["translations"][0]["translatedText"].asString();
    } else {
        cerr << "JSON Parsing Error: " << errors << endl;
    }

    return translatedText;
}

int main() {
    string apiKey = "YOUR_GOOGLE_TRANSLATE_API_KEY"; // Yahan apni API key daalain
    string text, sourceLang, targetLang;

    cout << "Enter text to translate: ";
    getline(cin, text);
    cout << "Enter source language (e.g., en): ";
    cin >> sourceLang;
    cout << "Enter target language (e.g., fr): ";
    cin >> targetLang;

    string translatedText = translateText(text, sourceLang, targetLang, apiKey);
    cout << "Translated Text: " << translatedText << endl;

    return 0;
}
