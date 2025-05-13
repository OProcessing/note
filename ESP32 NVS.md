## Summary
 `nvs_data.csv` 파일에 key value를 추가하고 build하여 출력하면 데이터를 얻을 수 있다.
``` csv
nvs_data.csv ==
radar_key1,data,u8,123
radar_key2,data,u8,124
radar_key3,data,u8,125

output >>
123
124
125
```

프로그램 상에서는 다음과 같이 read & write할 수 있다.
```
    uint8_t key1 = 0;
    nvs_get_u8(my_handle, "key1", &key1);

output >>
123

    key1 = 45; 
    nvs_set_u8(my_handle, "key1", key1);
    nvs_commit(&my_handle);
    
    key1 = 0;
    nvs_get_u8(my_handle, "key1", &key1);
    printf("%"PRIu8"\n", key1);

output >>
45
```

## esp32 NVS system
esp에는 간접적으로 flash memory를 사용할 수 있는 api를 제공한다. nvs api는 python의 dictionary처럼 key value 쌍으로 생성된다. esp-idf에서 제공하는 nvsgen project를 살펴보면 directory에 `nvs_data.csv` 파일이 있고, `CMakeLists.txt`에 `nvs_create_partition_image(nvs ../nvs_data.csv FLASH_IN_PROJECT)`이 추가되어있다. 프로젝트를 build하면 nvs_data.csv 파일에 저장된 데이터가 `partition_example.csv`에 지정한대로 `nvs,      data, nvs,     0x9000,  0x6000,`에 생성된다. `nvs_flash_init();`으로 nvs 시스템을 초기화하고 `nvs_handle_t my_handle;`과 같이 handle을 선언한다. `nvs_open_from_partition(name, namespace, mode, handle)`함수를 통해 호출할 partition, namespace, RW mode, handle을 설정한다. `nvs_get_i8(handle, key, value)`과 같은 함수로 데이터를 호출할 수 있다. 이 때 data type은 esp에서 제공하는 u8, i8, u16, i16, u32, i32, string type만 가능하다. 데이터를 저장하고 싶다면 `nvs_set_i8(handle, key, value);`를 사용하여 데이터를 등록하고, `nvs_commit(handle);`로 저장할 수 있다.
``` C
void app_main(void)
{
    // Initialize NVS
    esp_err_t err = nvs_flash_init();
    if (err == ESP_ERR_NVS_NO_FREE_PAGES || err == ESP_ERR_NVS_NEW_VERSION_FOUND) {
        // NVS partition was truncated and needs to be erased
        // Retry nvs_flash_init
        ESP_ERROR_CHECK(nvs_flash_erase());
        err = nvs_flash_init();
    }
    ESP_ERROR_CHECK(err);
    
    // Open the pre-filled NVS partition called "nvs"
    ESP_LOGI(TAG, "Opening Non-Volatile Storage (NVS) handle");
    nvs_handle_t my_handle;
    err = nvs_open_from_partition("nvs", "storage", NVS_READWRITE, &my_handle);
    if (err != ESP_OK) {
        ESP_LOGE(TAG, "Error (%s) opening NVS handle!\n", esp_err_to_name(err));
        return;
    }
    ESP_LOGI(TAG, "The NVS handle successfully opened");
       
    int8_t i8_val = 0;
    err = nvs_get_i8(my_handle, "i8_key", &i8_val);
    ESP_ERROR_CHECK(err);
    printf("%"PRIi8"\n", i8_val);
    assert(i8_val == -128);
    
    size_t str_len = 0;
    err = nvs_get_str(my_handle, "str_key", NULL, &str_len);
    ESP_ERROR_CHECK(err);
    assert(str_len == 222);

    char* str_val = (char*) malloc(str_len);
    err = nvs_get_str(my_handle, "str_key", str_val, &str_len);
    ESP_ERROR_CHECK(err);
    printf("%s\n", str_val);
    assert(str_val[0] == 'L');
    free(str_val);
      
    // Close
    nvs_close(my_handle);
    ESP_LOGI(TAG, "Reading values from NVS done - all OK");
}
    
```


## esp32 NVS function list
``` C
esp_err_t nvs_flash_init(void)
esp_err_t nvs_flash_init_partition(const char *partition_label)
esp_err_t nvs_flash_deinit(void)
esp_err_t nvs_flash_deinit_partition(const char *partition_label)
esp_err_t nvs_flash_erase(void)
esp_err_t nvs_flash_erase_partition(const char *part_name)
esp_err_t nvs_flash_secure_init(nvs_sec_cfg_t *cfg)
esp_err_t nvs_flash_secure_init_partition(const char *partition_label, nvs_sec_cfg_t *cfg)
esp_err_t nvs_flash_generate_keys(const esp_partition_t *partition, nvs_sec_cfg_t *cfg)
esp_err_t nvs_flash_read_security_cfg(const esp_partition_t *partition, nvs_sec_cfg_t *cfg)
esp_err_t nvs_flash_register_security_scheme(nvs_sec_scheme_t *scheme_cfg)
nvs_sec_scheme_t *nvs_flash_get_default_security_scheme(void)
esp_err_t nvs_flash_generate_keys_v2(nvs_sec_scheme_t *scheme_cfg, nvs_sec_cfg_t *cfg)
esp_err_t nvs_flash_read_security_cfg_v2(nvs_sec_scheme_t *scheme_cfg, nvs_sec_cfg_t *cfg)
esp_err_t nvs_set_i8(nvs_handle_t handle, const char *key, int8_t value)
esp_err_t nvs_get_i8(nvs_handle_t handle, const char *key, int8_t *out_value)
esp_err_t nvs_get_blob(nvs_handle_t handle, const char *key, void *out_value, size_t *length)
esp_err_t nvs_open(const char *namespace_name, nvs_open_mode_t open_mode, nvs_handle_t *out_handle)
esp_err_t nvs_open_from_partition(const char *part_name, const char *namespace_name, nvs_open_mode_t open_mode, nvs_handle_t *out_handle)
esp_err_t nvs_set_blob(nvs_handle_t handle, const char *key, const void *value, size_t length)
esp_err_t nvs_find_key(nvs_handle_t handle, const char *key, nvs_type_t *out_type)
esp_err_t nvs_erase_key(nvs_handle_t handle, const char *key)
esp_err_t nvs_erase_all(nvs_handle_t handle)
esp_err_t nvs_commit(nvs_handle_t handle)
void nvs_close(nvs_handle_t handle)
esp_err_t nvs_get_stats(const char *part_name, nvs_stats_t *nvs_stats)
esp_err_t nvs_get_used_entry_count(nvs_handle_t handle, size_t *used_entries)
esp_err_t nvs_entry_find(const char *part_name, const char *namespace_name, nvs_type_t type, nvs_iterator_t *output_iterator)
esp_err_t nvs_entry_find_in_handle(nvs_handle_t handle, nvs_type_t type, nvs_iterator_t *output_iterator)
esp_err_t nvs_entry_next(nvs_iterator_t *iterator)
esp_err_t nvs_entry_info(const nvs_iterator_t iterator, nvs_entry_info_t *out_info)
void nvs_release_iterator(nvs_iterator_t iterator)
```
