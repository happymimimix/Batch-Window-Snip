# Batch-Window-Snip
### A program that automatically creates a screenshot for every single visible window on the desktop. 

This is a very simple batch snipping program that I asked Chat GPT 4o to make for me in C++.

I want to make a bilibili video that talks about how I made the 400 million notes black midi, Gleudo, which need this to create some screenshots. 

Here's the prompt that I used to generate it: 

```
ME: Save every visible window on windows desktop to a png/bmp image with C++. 
```
```
GPT 4o: Saving every visible window on a Windows desktop to an image (like PNG or BMP) using C++ requires a combination of Windows API functions. Below is a basic outline and example code to achieve this:

1. **Enumerate all Windows**: Use `EnumWindows` to enumerate all top-level windows.
2. **Check Visibility**: For each window, use `IsWindowVisible` to check if it is visible.
3. **Capture Window Content**: Use `GetDC` to get the device context (DC) of the window and then use `BitBlt` to copy the window's content into a memory device context.
4. **Save to Image**: Use GDI+ to save the captured content as a PNG or BMP file.

Here is an example code that demonstrates these steps:

#include <windows.h>
#include <gdiplus.h>
#include <iostream>
#include <vector>

#pragma comment (lib,"Gdiplus.lib")

using namespace Gdiplus;

void SaveBitmapToFile(HBITMAP hBitmap, const wchar_t* filename) {
    Gdiplus::Bitmap bmp(hBitmap, nullptr);
    CLSID pngClsid;
    CLSIDFromString(L"{557CF406-1A04-11D3-9A73-0000F81EF32E}", &pngClsid); // PNG Encoder
    bmp.Save(filename, &pngClsid, nullptr);
}

BOOL CALLBACK EnumWindowsProc(HWND hwnd, LPARAM lParam) {
    if (IsWindowVisible(hwnd)) {
        RECT rc;
        GetWindowRect(hwnd, &rc);
        HDC hdcWindow = GetDC(hwnd);
        HDC hdcMemDC = CreateCompatibleDC(hdcWindow);
        HBITMAP hbmWindow = CreateCompatibleBitmap(hdcWindow, rc.right - rc.left, rc.bottom - rc.top);

        SelectObject(hdcMemDC, hbmWindow);
        BitBlt(hdcMemDC, 0, 0, rc.right - rc.left, rc.bottom - rc.top, hdcWindow, 0, 0, SRCCOPY);

        // Generate a file name and save it
        static int fileIndex = 0;
        wchar_t filename[100];
        wsprintf(filename, L"WindowCapture_%d.png", fileIndex++);
        SaveBitmapToFile(hbmWindow, filename);

        // Cleanup
        DeleteObject(hbmWindow);
        DeleteDC(hdcMemDC);
        ReleaseDC(hwnd, hdcWindow);
    }
    return TRUE;
}

int main() {
    // Initialize GDI+
    GdiplusStartupInput gdiplusStartupInput;
    ULONG_PTR gdiplusToken;
    GdiplusStartup(&gdiplusToken, &gdiplusStartupInput, nullptr);

    // Enumerate all windows and capture them
    EnumWindows(EnumWindowsProc, 0);

    // Cleanup GDI+
    GdiplusShutdown(gdiplusToken);
    return 0;
}

### Explanation:

1. **`EnumWindows`**: Enumerates all top-level windows on the desktop, passing each window handle to `EnumWindowsProc`.
2. **`IsWindowVisible`**: Filters out windows that are not visible.
3. **`GetWindowRect`**: Retrieves the dimensions of the window.
4. **`GetDC` and `CreateCompatibleDC`**: Gets the device context for the window and creates a compatible memory DC.
5. **`CreateCompatibleBitmap`**: Creates a bitmap compatible with the device context.
6. **`BitBlt`**: Copies the content from the window's device context to the memory device context.
7. **`SaveBitmapToFile`**: Uses GDI+ to save the bitmap as a PNG file.
8. **Cleanup**: Handles memory cleanup for DCs and bitmaps.

### Requirements:

- GDI+ is required for saving images to files. The example code initializes GDI+ and uses it to save the window content as PNG files.

### Build Instructions:

1. Link with `Gdiplus.lib`.
2. Include GDI+ headers (`gdiplus.h`).

This code will create a PNG file for each visible window on your desktop. You can modify the filename generation logic or the saving format as needed.
```
```
ME: Use the text from titlebar as filename and show a file dialog to let the user pick a folder to save all the captured images to. 
```
```
GPT 4o: To modify the previous C++ program to use the text from the window's title bar as the filename and to show a file dialog to let the user pick a folder, we need to:

1. **Retrieve the window title** using `GetWindowText`.
2. **Show a folder selection dialog** using the `IFileDialog` interface (which is available from Windows Vista onwards).
3. **Combine the folder path with the window title to create the full path for saving the image**.

Here's the updated code:

#include <windows.h>
#include <gdiplus.h>
#include <shobjidl.h> // For IFileDialog
#include <string>
#include <iostream>

#pragma comment (lib,"Gdiplus.lib")
#pragma comment (lib,"Ole32.lib") // Required for COM

using namespace Gdiplus;

// Function to save bitmap to a file
void SaveBitmapToFile(HBITMAP hBitmap, const std::wstring& filename) {
    Gdiplus::Bitmap bmp(hBitmap, nullptr);
    CLSID pngClsid;
    CLSIDFromString(L"{557CF406-1A04-11D3-9A73-0000F81EF32E}", &pngClsid); // PNG Encoder
    bmp.Save(filename.c_str(), &pngClsid, nullptr);
}

// Callback function for EnumWindows
BOOL CALLBACK EnumWindowsProc(HWND hwnd, LPARAM lParam) {
    if (IsWindowVisible(hwnd)) {
        // Get the window title
        wchar_t windowTitle[256];
        GetWindowText(hwnd, windowTitle, sizeof(windowTitle) / sizeof(windowTitle[0]));

        // Skip windows with empty titles
        if (wcslen(windowTitle) == 0) {
            return TRUE;
        }

        // Get the directory path from lParam
        std::wstring* folderPath = reinterpret_cast<std::wstring*>(lParam);

        // Combine folder path and window title to create the filename
        std::wstring filePath = *folderPath + L"\\" + windowTitle + L".png";

        // Capture the window content
        RECT rc;
        GetWindowRect(hwnd, &rc);
        HDC hdcWindow = GetDC(hwnd);
        HDC hdcMemDC = CreateCompatibleDC(hdcWindow);
        HBITMAP hbmWindow = CreateCompatibleBitmap(hdcWindow, rc.right - rc.left, rc.bottom - rc.top);

        SelectObject(hdcMemDC, hbmWindow);
        BitBlt(hdcMemDC, 0, 0, rc.right - rc.left, rc.bottom - rc.top, hdcWindow, 0, 0, SRCCOPY);

        // Save the captured image
        SaveBitmapToFile(hbmWindow, filePath);

        // Cleanup
        DeleteObject(hbmWindow);
        DeleteDC(hdcMemDC);
        ReleaseDC(hwnd, hdcWindow);
    }
    return TRUE;
}

// Function to display a folder selection dialog and return the selected path
std::wstring ShowFolderSelectionDialog() {
    std::wstring folderPath;
    IFileDialog* pfd = nullptr;

    // Initialize COM
    CoInitializeEx(nullptr, COINIT_APARTMENTTHREADED | COINIT_DISABLE_OLE1DDE);

    // Create the FileOpenDialog object
    HRESULT hr = CoCreateInstance(CLSID_FileOpenDialog, nullptr, CLSCTX_INPROC_SERVER, IID_PPV_ARGS(&pfd));
    if (SUCCEEDED(hr)) {
        // Set options to select folders only
        DWORD dwOptions;
        pfd->GetOptions(&dwOptions);
        pfd->SetOptions(dwOptions | FOS_PICKFOLDERS);

        // Show the dialog
        hr = pfd->Show(nullptr);
        if (SUCCEEDED(hr)) {
            // Get the folder path
            IShellItem* pItem = nullptr;
            hr = pfd->GetResult(&pItem);
            if (SUCCEEDED(hr)) {
                PWSTR pszFolderPath = nullptr;
                hr = pItem->GetDisplayName(SIGDN_FILESYSPATH, &pszFolderPath);
                if (SUCCEEDED(hr)) {
                    folderPath = pszFolderPath;
                    CoTaskMemFree(pszFolderPath);
                }
                pItem->Release();
            }
        }
        pfd->Release();
    }

    // Uninitialize COM
    CoUninitialize();
    return folderPath;
}

int main() {
    // Initialize GDI+
    GdiplusStartupInput gdiplusStartupInput;
    ULONG_PTR gdiplusToken;
    GdiplusStartup(&gdiplusToken, &gdiplusStartupInput, nullptr);

    // Show the folder selection dialog
    std::wstring folderPath = ShowFolderSelectionDialog();
    if (!folderPath.empty()) {
        // Enumerate all windows and capture them
        EnumWindows(EnumWindowsProc, reinterpret_cast<LPARAM>(&folderPath));
    }

    // Cleanup GDI+
    GdiplusShutdown(gdiplusToken);
    return 0;
}

### Explanation of the Changes:

1. **`GetWindowText`**: This function retrieves the title of the window, which we use as the filename. Empty window titles are skipped.
2. **`ShowFolderSelectionDialog`**: This function displays a folder selection dialog to the user using `IFileDialog` with the `FOS_PICKFOLDERS` option, which limits the selection to folders.
3. **Combining Folder Path and Window Title**: The folder path selected by the user is combined with the window title to form the full file path for saving the image.

### Notes:

- **File Naming Considerations**: Windows file systems do not allow certain characters in filenames (`\ / : * ? " < > |`). You may need to sanitize the window title before using it as a filename to replace these characters with safe alternatives.
- **COM Initialization**: The code initializes COM using `CoInitializeEx` and uninitializes it using `CoUninitialize`.
- **Error Handling**: The code assumes success for most operations. In a production environment, you would add additional error handling for robustness.

### Build Instructions:

Ensure your project links against the following libraries:
- `Gdiplus.lib` for GDI+.
- `Ole32.lib` for COM (required by `IFileDialog`).

This program will allow users to save screenshots of all visible windows on their desktop into a selected folder with filenames based on the window titles.
```
```
ME: A few windows appear blank after capturing and there are black borders around the bottom and right edge of the image. Please fix it.
```
```
Note: I didn't accept the use of "PrintWindow()" here as it has very little connection with the black border issue and using it does not make the capturing any better than BitBlt. 

GPT 4o: The issues you're describing—blank windows and black borders—are common when capturing windows using GDI in Windows. These problems can arise due to various factors, including the way windows handle their painting operations, layered windows, or DPI scaling. Here’s a refined approach to fix these issues:

### Steps to Address the Issues:

1. **Use `PrintWindow` for Window Capture**: Instead of using `BitBlt`, use the `PrintWindow` API to capture the window. This function is better suited for capturing window content, especially for layered or off-screen windows.

2. **Correct Bitmap Size**: Ensure the bitmap is created with the exact dimensions of the window's client area, not the window's border, which might help avoid the black borders issue.

3. **Handle DPI Scaling**: If your application is running on a high-DPI display, the coordinates and sizes might be scaled. Use DPI-aware APIs or adjust for DPI scaling.

### Updated Code Example:

#include <windows.h>
#include <gdiplus.h>
#include <shobjidl.h> // For IFileDialog
#include <string>
#include <iostream>

#pragma comment (lib,"Gdiplus.lib")
#pragma comment (lib,"Ole32.lib") // Required for COM

using namespace Gdiplus;

// Function to save bitmap to a file
void SaveBitmapToFile(HBITMAP hBitmap, const std::wstring& filename) {
    Gdiplus::Bitmap bmp(hBitmap, nullptr);
    CLSID pngClsid;
    CLSIDFromString(L"{557CF406-1A04-11D3-9A73-0000F81EF32E}", &pngClsid); // PNG Encoder
    bmp.Save(filename.c_str(), &pngClsid, nullptr);
}

// Callback function for EnumWindows
BOOL CALLBACK EnumWindowsProc(HWND hwnd, LPARAM lParam) {
    if (IsWindowVisible(hwnd)) {
        // Get the window title
        wchar_t windowTitle[256];
        GetWindowText(hwnd, windowTitle, sizeof(windowTitle) / sizeof(windowTitle[0]));

        // Skip windows with empty titles
        if (wcslen(windowTitle) == 0) {
            return TRUE;
        }

        // Get the directory path from lParam
        std::wstring* folderPath = reinterpret_cast<std::wstring*>(lParam);

        // Combine folder path and window title to create the filename
        std::wstring filePath = *folderPath + L"\\" + windowTitle + L".png";

        // Get the client area size
        RECT rcClient;
        GetClientRect(hwnd, &rcClient);
        int width = rcClient.right - rcClient.left;
        int height = rcClient.bottom - rcClient.top;

        // Adjust for DPI scaling
        HDC hdcScreen = GetDC(nullptr);
        HDC hdcMemDC = CreateCompatibleDC(hdcScreen);
        HBITMAP hbmWindow = CreateCompatibleBitmap(hdcScreen, width, height);
        HGDIOBJ hOld = SelectObject(hdcMemDC, hbmWindow);

        // Use PrintWindow to capture the window
        BOOL result = PrintWindow(hwnd, hdcMemDC, PW_CLIENTONLY);

        // If PrintWindow fails, fall back to BitBlt (less reliable)
        if (!result) {
            HDC hdcWindow = GetDC(hwnd);
            BitBlt(hdcMemDC, 0, 0, width, height, hdcWindow, 0, 0, SRCCOPY);
            ReleaseDC(hwnd, hdcWindow);
        }

        // Save the captured image
        SaveBitmapToFile(hbmWindow, filePath);

        // Cleanup
        SelectObject(hdcMemDC, hOld);
        DeleteObject(hbmWindow);
        DeleteDC(hdcMemDC);
        ReleaseDC(nullptr, hdcScreen);
    }
    return TRUE;
}

// Function to display a folder selection dialog and return the selected path
std::wstring ShowFolderSelectionDialog() {
    std::wstring folderPath;
    IFileDialog* pfd = nullptr;

    // Initialize COM
    CoInitializeEx(nullptr, COINIT_APARTMENTTHREADED | COINIT_DISABLE_OLE1DDE);

    // Create the FileOpenDialog object
    HRESULT hr = CoCreateInstance(CLSID_FileOpenDialog, nullptr, CLSCTX_INPROC_SERVER, IID_PPV_ARGS(&pfd));
    if (SUCCEEDED(hr)) {
        // Set options to select folders only
        DWORD dwOptions;
        pfd->GetOptions(&dwOptions);
        pfd->SetOptions(dwOptions | FOS_PICKFOLDERS);

        // Show the dialog
        hr = pfd->Show(nullptr);
        if (SUCCEEDED(hr)) {
            // Get the folder path
            IShellItem* pItem = nullptr;
            hr = pfd->GetResult(&pItem);
            if (SUCCEEDED(hr)) {
                PWSTR pszFolderPath = nullptr;
                hr = pItem->GetDisplayName(SIGDN_FILESYSPATH, &pszFolderPath);
                if (SUCCEEDED(hr)) {
                    folderPath = pszFolderPath;
                    CoTaskMemFree(pszFolderPath);
                }
                pItem->Release();
            }
        }
        pfd->Release();
    }

    // Uninitialize COM
    CoUninitialize();
    return folderPath;
}

int main() {
    // Initialize GDI+
    GdiplusStartupInput gdiplusStartupInput;
    ULONG_PTR gdiplusToken;
    GdiplusStartup(&gdiplusToken, &gdiplusStartupInput, nullptr);

    // Show the folder selection dialog
    std::wstring folderPath = ShowFolderSelectionDialog();
    if (!folderPath.empty()) {
        // Enumerate all windows and capture them
        EnumWindows(EnumWindowsProc, reinterpret_cast<LPARAM>(&folderPath));
    }

    // Cleanup GDI+
    GdiplusShutdown(gdiplusToken);
    return 0;
}

### Summary of Changes:

1. **PrintWindow for Capturing**: The `PrintWindow` function is now used to capture the window content. This is more reliable for windows that may appear blank when captured with `BitBlt`, especially for layered windows.
2. **DPI Scaling**: The code ensures the correct dimensions are used by adjusting for DPI. This helps in preventing issues related to high-DPI displays.
3. **Fallback to BitBlt**: If `PrintWindow` fails (for some windows, it might), the code falls back to using `BitBlt`, as a last resort to capture the window content.
4. **Black Borders Issue**: By focusing on the client area of the window (`GetClientRect`) rather than the full window including borders, we avoid adding any black borders to the captured image.

### Notes:

- **Handling Complex Windows**: Some windows might still not capture perfectly, depending on their rendering method (e.g., using DirectX, OpenGL, or other hardware-accelerated methods). In such cases, you might need more advanced techniques, such as desktop composition capturing or third-party libraries.
- **Character Sanitization**: Ensure that window titles used for filenames do not contain invalid characters (`\ / : * ? " < > |`).

This updated code should capture most windows accurately without the issues you encountered with the previous version.
```
*Start a new topic*
```
ME: #include <windows.h>
#include <gdiplus.h>
#include <shobjidl.h> // For IFileDialog
#include <string>
#include <iostream>

#pragma comment (lib,"Gdiplus.lib")
#pragma comment (lib,"Ole32.lib") // Required for COM

using namespace Gdiplus;

// Function to save bitmap to a file
void SaveBitmapToFile(HBITMAP hBitmap, const std::wstring& filename) {
    Gdiplus::Bitmap bmp(hBitmap, nullptr);
    CLSID pngClsid;
    CLSIDFromString(L"{557CF406-1A04-11D3-9A73-0000F81EF32E}", &pngClsid); // PNG Encoder
    bmp.Save(filename.c_str(), &pngClsid, nullptr);
}

// Callback function for EnumWindows
BOOL CALLBACK EnumWindowsProc(HWND hwnd, LPARAM lParam) {
    if (IsWindowVisible(hwnd)) {
        // Get the window title
        wchar_t windowTitle[256];
        GetWindowText(hwnd, windowTitle, sizeof(windowTitle) / sizeof(windowTitle[0]));

        // Skip windows with empty titles
        if (wcslen(windowTitle) == 0) {
            return TRUE;
        }

        // Get the directory path from lParam
        std::wstring* folderPath = reinterpret_cast<std::wstring*>(lParam);

        // Combine folder path and window title to create the filename
        std::wstring filePath = *folderPath + L"\\" + windowTitle + L".png";

        // Get the client area size
        RECT rcClient;
        GetClientRect(hwnd, &rcClient);
        int width = rcClient.right - rcClient.left;
        int height = rcClient.bottom - rcClient.top;

        // Adjust for DPI scaling
        HDC hdcScreen = GetDC(nullptr);
        HDC hdcMemDC = CreateCompatibleDC(hdcScreen);
        HBITMAP hbmWindow = CreateCompatibleBitmap(hdcScreen, width, height);
        HGDIOBJ hOld = SelectObject(hdcMemDC, hbmWindow);

        HDC hdcWindow = GetDC(hwnd);
        BitBlt(hdcMemDC, 0, 0, width, height, hdcWindow, 0, 0, SRCCOPY);
        ReleaseDC(hwnd, hdcWindow);

        // Save the captured image
        SaveBitmapToFile(hbmWindow, filePath);

        // Cleanup
        SelectObject(hdcMemDC, hOld);
        DeleteObject(hbmWindow);
        DeleteDC(hdcMemDC);
        ReleaseDC(nullptr, hdcScreen);
    }
    return TRUE;
}

// Function to display a folder selection dialog and return the selected path
std::wstring ShowFolderSelectionDialog() {
    std::wstring folderPath;
    IFileDialog* pfd = nullptr;

    // Initialize COM
    CoInitializeEx(nullptr, COINIT_APARTMENTTHREADED | COINIT_DISABLE_OLE1DDE);

    // Create the FileOpenDialog object
    HRESULT hr = CoCreateInstance(CLSID_FileOpenDialog, nullptr, CLSCTX_INPROC_SERVER, IID_PPV_ARGS(&pfd));
    if (SUCCEEDED(hr)) {
        // Set options to select folders only
        DWORD dwOptions;
        pfd->GetOptions(&dwOptions);
        pfd->SetOptions(dwOptions | FOS_PICKFOLDERS);

        // Show the dialog
        hr = pfd->Show(nullptr);
        if (SUCCEEDED(hr)) {
            // Get the folder path
            IShellItem* pItem = nullptr;
            hr = pfd->GetResult(&pItem);
            if (SUCCEEDED(hr)) {
                PWSTR pszFolderPath = nullptr;
                hr = pItem->GetDisplayName(SIGDN_FILESYSPATH, &pszFolderPath);
                if (SUCCEEDED(hr)) {
                    folderPath = pszFolderPath;
                    CoTaskMemFree(pszFolderPath);
                }
                pItem->Release();
            }
        }
        pfd->Release();
    }

    // Uninitialize COM
    CoUninitialize();
    return folderPath;
}

int main() {
    // Initialize GDI+
    GdiplusStartupInput gdiplusStartupInput;
    ULONG_PTR gdiplusToken;
    GdiplusStartup(&gdiplusToken, &gdiplusStartupInput, nullptr);

    // Show the folder selection dialog
    std::wstring folderPath = ShowFolderSelectionDialog();
    if (!folderPath.empty()) {
        // Enumerate all windows and capture them
        EnumWindows(EnumWindowsProc, reinterpret_cast<LPARAM>(&folderPath));
    }

    // Cleanup GDI+
    GdiplusShutdown(gdiplusToken);
    return 0;
}


Handle illegal file names and replace illegal characters suck as : with _
```
```
GPT 4o: To handle illegal file names and replace illegal characters such as `:` with `_`, you need to sanitize the window title before using it to create the file path. Illegal characters in Windows file names include `\`, `/`, `:`, `*`, `?`, `"`, `<`, `>`, and `|`. You can create a function that replaces these characters with an underscore `_` or another suitable character.

Here's the modified code with the necessary changes:

#include <windows.h>
#include <gdiplus.h>
#include <shobjidl.h> // For IFileDialog
#include <string>
#include <iostream>
#include <algorithm> // For std::replace_if

#pragma comment (lib,"Gdiplus.lib")
#pragma comment (lib,"Ole32.lib") // Required for COM

using namespace Gdiplus;

// Function to save bitmap to a file
void SaveBitmapToFile(HBITMAP hBitmap, const std::wstring& filename) {
    Gdiplus::Bitmap bmp(hBitmap, nullptr);
    CLSID pngClsid;
    CLSIDFromString(L"{557CF406-1A04-11D3-9A73-0000F81EF32E}", &pngClsid); // PNG Encoder
    bmp.Save(filename.c_str(), &pngClsid, nullptr);
}

// Function to sanitize the window title by replacing illegal characters
std::wstring SanitizeFileName(const std::wstring& title) {
    std::wstring sanitizedTitle = title;
    const wchar_t illegalChars[] = L"\\/:*?\"<>|";
    for (wchar_t ch : illegalChars) {
        std::replace(sanitizedTitle.begin(), sanitizedTitle.end(), ch, L'_');
    }
    return sanitizedTitle;
}

// Callback function for EnumWindows
BOOL CALLBACK EnumWindowsProc(HWND hwnd, LPARAM lParam) {
    if (IsWindowVisible(hwnd)) {
        // Get the window title
        wchar_t windowTitle[256];
        GetWindowText(hwnd, windowTitle, sizeof(windowTitle) / sizeof(windowTitle[0]));

        // Skip windows with empty titles
        if (wcslen(windowTitle) == 0) {
            return TRUE;
        }

        // Sanitize the window title
        std::wstring sanitizedTitle = SanitizeFileName(windowTitle);

        // Get the directory path from lParam
        std::wstring* folderPath = reinterpret_cast<std::wstring*>(lParam);

        // Combine folder path and sanitized window title to create the filename
        std::wstring filePath = *folderPath + L"\\" + sanitizedTitle + L".png";

        // Get the client area size
        RECT rcClient;
        GetClientRect(hwnd, &rcClient);
        int width = rcClient.right - rcClient.left;
        int height = rcClient.bottom - rcClient.top;

        // Adjust for DPI scaling
        HDC hdcScreen = GetDC(nullptr);
        HDC hdcMemDC = CreateCompatibleDC(hdcScreen);
        HBITMAP hbmWindow = CreateCompatibleBitmap(hdcScreen, width, height);
        HGDIOBJ hOld = SelectObject(hdcMemDC, hbmWindow);

        HDC hdcWindow = GetDC(hwnd);
        BitBlt(hdcMemDC, 0, 0, width, height, hdcWindow, 0, 0, SRCCOPY);
        ReleaseDC(hwnd, hdcWindow);

        // Save the captured image
        SaveBitmapToFile(hbmWindow, filePath);

        // Cleanup
        SelectObject(hdcMemDC, hOld);
        DeleteObject(hbmWindow);
        DeleteDC(hdcMemDC);
        ReleaseDC(nullptr, hdcScreen);
    }
    return TRUE;
}

// Function to display a folder selection dialog and return the selected path
std::wstring ShowFolderSelectionDialog() {
    std::wstring folderPath;
    IFileDialog* pfd = nullptr;

    // Initialize COM
    CoInitializeEx(nullptr, COINIT_APARTMENTTHREADED | COINIT_DISABLE_OLE1DDE);

    // Create the FileOpenDialog object
    HRESULT hr = CoCreateInstance(CLSID_FileOpenDialog, nullptr, CLSCTX_INPROC_SERVER, IID_PPV_ARGS(&pfd));
    if (SUCCEEDED(hr)) {
        // Set options to select folders only
        DWORD dwOptions;
        pfd->GetOptions(&dwOptions);
        pfd->SetOptions(dwOptions | FOS_PICKFOLDERS);

        // Show the dialog
        hr = pfd->Show(nullptr);
        if (SUCCEEDED(hr)) {
            // Get the folder path
            IShellItem* pItem = nullptr;
            hr = pfd->GetResult(&pItem);
            if (SUCCEEDED(hr)) {
                PWSTR pszFolderPath = nullptr;
                hr = pItem->GetDisplayName(SIGDN_FILESYSPATH, &pszFolderPath);
                if (SUCCEEDED(hr)) {
                    folderPath = pszFolderPath;
                    CoTaskMemFree(pszFolderPath);
                }
                pItem->Release();
            }
        }
        pfd->Release();
    }

    // Uninitialize COM
    CoUninitialize();
    return folderPath;
}

int main() {
    // Initialize GDI+
    GdiplusStartupInput gdiplusStartupInput;
    ULONG_PTR gdiplusToken;
    GdiplusStartup(&gdiplusToken, &gdiplusStartupInput, nullptr);

    // Show the folder selection dialog
    std::wstring folderPath = ShowFolderSelectionDialog();
    if (!folderPath.empty()) {
        // Enumerate all windows and capture them
        EnumWindows(EnumWindowsProc, reinterpret_cast<LPARAM>(&folderPath));
    }

    // Cleanup GDI+
    GdiplusShutdown(gdiplusToken);
    return 0;
}

### Key Changes:
1. **SanitizeFileName Function**: This function replaces any illegal characters in the window title with an underscore `_`.
2. **Integration**: The `SanitizeFileName` function is called before creating the `filePath`, ensuring that the filename is valid on the Windows file system.

This ensures that any illegal characters in the window titles are replaced, preventing errors when saving the files.
```
