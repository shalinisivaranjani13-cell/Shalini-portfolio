// fetch_site.c
// Compile: gcc -o fetch_site fetch_site.c -lcurl
// Usage: ./fetch_site
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <curl/curl.h>

struct WriteData {
    FILE *fp;
};

static size_t write_callback(void *ptr, size_t size, size_t nmemb, void *userdata) {
    struct WriteData *wd = (struct WriteData *)userdata;
    size_t written = fwrite(ptr, size, nmemb, wd->fp);
    return written;
}

int main(void) {
    const char *url = "https://shalini-personal-por-e9uw.bolt.host/";
    const char *out_filename = "index.html";

    CURL *curl;
    CURLcode res;
    struct WriteData wd;

    wd.fp = fopen(out_filename, "wb");
    if (!wd.fp) {
        fprintf(stderr, "Error: cannot open %s for writing\n", out_filename);
        return 1;
    }

    curl = curl_easy_init();
    if (!curl) {
        fprintf(stderr, "Error: curl_easy_init() failed\n");
        fclose(wd.fp);
        return 1;
    }

    curl_easy_setopt(curl, CURLOPT_URL, url);
    curl_easy_setopt(curl, CURLOPT_FOLLOWLOCATION, 1L);     // follow redirects
    curl_easy_setopt(curl, CURLOPT_WRITEFUNCTION, write_callback);
    curl_easy_setopt(curl, CURLOPT_WRITEDATA, &wd);
    curl_easy_setopt(curl, CURLOPT_USERAGENT, "fetch_site/1.0");

    // Optional: set reasonable timeouts
    curl_easy_setopt(curl, CURLOPT_TIMEOUT, 30L);
    curl_easy_setopt(curl, CURLOPT_CONNECTTIMEOUT, 10L);

    res = curl_easy_perform(curl);
    if (res != CURLE_OK) {
        fprintf(stderr, "curl_easy_perform() failed: %s\n", curl_easy_strerror(res));
        fclose(wd.fp);
        curl_easy_cleanup(curl);
        return 1;
    }

    // Clean up
    curl_easy_cleanup(curl);
    fclose(wd.fp);

    printf("Saved page to %s\n", out_filename);
    return 0;
}
