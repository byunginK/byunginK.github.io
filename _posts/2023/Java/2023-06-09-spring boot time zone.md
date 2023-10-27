---
layout: post
title: "Spring boot timeZone"
date: 2023-06-09
categories: [JAVA]
---

# Java - Display all ZoneId and its UTC offset

A Java 8 example to display all the ZoneId and its OffSet hours and minutes.

*P.S Tested with Java 8 and 12*

## 1. Display ZoneId and Offset

DisplayZoneAndOffSet.java

```java
import java.time.LocalDateTime;
import java.time.ZoneId;
import java.time.ZoneOffset;
import java.time.ZonedDateTime;
import java.util.HashMap;
import java.util.LinkedHashMap;
import java.util.Map;

public class DisplayZoneAndOffSet {

    public static final boolean SORT_BY_REGION = false;

    public static void main(String[] argv) {

        Map<String, String> sortedMap = new LinkedHashMap<>();

        Map<String, String> allZoneIdsAndItsOffSet = getAllZoneIdsAndItsOffSet();

        //sort map by key
        if (SORT_BY_REGION) {
            allZoneIdsAndItsOffSet.entrySet().stream()
                    .sorted(Map.Entry.comparingByKey())
                    .forEachOrdered(e -> sortedMap.put(e.getKey(), e.getValue()));
        } else {
            // sort by value, descending order
            allZoneIdsAndItsOffSet.entrySet().stream()
                    .sorted(Map.Entry.<String, String>comparingByValue().reversed())
                    .forEachOrdered(e -> sortedMap.put(e.getKey(), e.getValue()));
        }

        // print map
        sortedMap.forEach((k, v) ->
        {
            String out = String.format("%35s (UTC%s) %n", k, v);
            System.out.printf(out);
        });

        System.out.println("\nTotal Zone IDs " + sortedMap.size());

    }

    private static Map<String, String> getAllZoneIdsAndItsOffSet() {

        Map<String, String> result = new HashMap<>();

        LocalDateTime localDateTime = LocalDateTime.now();

        for (String zoneId : ZoneId.getAvailableZoneIds()) {

            ZoneId id = ZoneId.of(zoneId);

            // LocalDateTime -> ZonedDateTime
            ZonedDateTime zonedDateTime = localDateTime.atZone(id);

            // ZonedDateTime -> ZoneOffset
            ZoneOffset zoneOffset = zonedDateTime.getOffset();

            //replace Z to +00:00
            String offset = zoneOffset.getId().replaceAll("Z", "+00:00");

            result.put(id.toString(), offset);

        }

        return result;

    }

}
```

출력
```

                         Etc/GMT+12 (UTC-12:00)
                  Pacific/Pago_Pago (UTC-11:00)
                      Pacific/Samoa (UTC-11:00)
                       Pacific/Niue (UTC-11:00)
                           US/Samoa (UTC-11:00)
                         Etc/GMT+11 (UTC-11:00)
                     Pacific/Midway (UTC-11:00)
                   Pacific/Honolulu (UTC-10:00)
                  Pacific/Rarotonga (UTC-10:00)
                     Pacific/Tahiti (UTC-10:00)
                   Pacific/Johnston (UTC-10:00)
                          US/Hawaii (UTC-10:00)
                      SystemV/HST10 (UTC-10:00)
                         Etc/GMT+10 (UTC-10:00)
                  Pacific/Marquesas (UTC-09:30)
                          Etc/GMT+9 (UTC-09:00)
                    Pacific/Gambier (UTC-09:00)
                       America/Atka (UTC-09:00)
                       SystemV/YST9 (UTC-09:00)
                       America/Adak (UTC-09:00)
                        US/Aleutian (UTC-09:00)
                          Etc/GMT+8 (UTC-08:00)
                          US/Alaska (UTC-08:00)
                     America/Juneau (UTC-08:00)
                 America/Metlakatla (UTC-08:00)
                    America/Yakutat (UTC-08:00)
                   Pacific/Pitcairn (UTC-08:00)
                      America/Sitka (UTC-08:00)
                  America/Anchorage (UTC-08:00)
                       SystemV/PST8 (UTC-08:00)
					   ... 계속

총 599개					   
```