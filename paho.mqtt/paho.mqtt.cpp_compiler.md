# paho.mqtt.cpp交叉编译

首先编译paho.mqtt.c
>cd paho.mqtt.c \
>mkdir build_arm \
>cd build_arm \
>cmake ..  -DCMAKE_C_COMPILER=arm-linux-gnueabihf-gcc  \
-DCMAKE_INSTALL_PREFIX=./install  \
-DPAHO_WITH_SSL=TRUE  \
-DOPENSSL_SSL_LIBRARY=/home/rising/repo/openssl_1.0.0_arm/lib/libssl.so 	\
-DOPENSSL_INCLUDE_DIR=/home/rising/repo/openssl_1.0.0_arm/include 	\
-DOPENSSL_CRYPTO_LIBRARY=/home/rising/repo/openssl_1.0.0_arm/lib/libcrypto.so \
-DCMAKE_BUILD_TYPE=Debug \
>#make \
>#make install 

再编译paho.mqtt.cpp
>cd paho.mqtt.cpp \
>mkdir build_arm \
>cd build_arm \
>cmake ..  -DCMAKE_CXX_COMPILER=arm-linux-gnueabihf-g++  \
-DCMAKE_INSTALL_PREFIX=./install  \
-DPAHO_MQTT_C_LIBRARIES=/home/rising/repo/paho.mqtt.c/build_arm/install/lib/libpaho-mqtt3c.so.1.3.8  \
-DPAHO_MQTT_C_INCLUDE_DIRS=/home/rising/repo/paho.mqtt.c/build_arm/install/include  \
-DOPENSSL_SSL_LIBRARY=/home/rising/repo/openssl_1.0.0_arm/lib/libssl.so  \
-DOPENSSL_INCLUDE_DIR=/home/rising/repo/openssl_1.0.0_arm/include  \
-DOPENSSL_CRYPTO_LIBRARY=/home/rising/repo/openssl_1.0.0_arm/lib/libcrypto.so \
-DCMAKE_BUILD_TYPE=Debug \
>make \
>make install



