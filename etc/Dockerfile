FROM tiryoh/ros-desktop-vnc:noetic-amd64-20240512T0225

# ROOTにNoeticの環境をインストールする
RUN /tmp/ros_setup_scripts_ubuntu/ros-noetic-desktop.sh

# 必要な依存関係のインストール
RUN apt-get update && apt-get install -y \
    curl \
    git \
    python3-pip \
    ros-noetic-usb-cam \
    ros-noetic-cv-bridge \
    sudo \
    vim \
    expect \
    && rm -rf /var/lib/apt/lists/*

# キーボード設定を事前に設定
RUN echo 'keyboard-configuration keyboard-configuration/layout select English (US)' | debconf-set-selections
RUN echo 'keyboard-configuration keyboard-configuration/layoutcode select "us"' | debconf-set-selections

# XSArmの制御用の各種機能をダウンロード＆インストール
RUN curl 'https://raw.githubusercontent.com/Interbotix/interbotix_ros_manipulators/main/interbotix_ros_xsarms/install/amd64/xsarm_amd64_install.sh' > xsarm_amd64_install.sh && \
    chmod +x xsarm_amd64_install.sh && \
    ./xsarm_amd64_install.sh -d noetic -n -p /root/interbotix_ws

# MobileALOHAのリポジトリをクローン
RUN cd /root/interbotix_ws/src && git clone https://github.com/MarkFzp/mobile-aloha

# ワークスペースのビルド
RUN /bin/bash -c '. /opt/ros/noetic/setup.sh; /root/interbotix_ws/devel/setup.sh; cd /root/interbotix_ws; catkin_make'

# interbotix_xs_toolboxのarm.pyファイルを編集
RUN sed -i '/self.T_sb = mr.FKinSpace(self.robot_des.M, self.robot_des.Slist, self.joint_commands)/c\        self.T_sb = None' /root/interbotix_ws/src/interbotix_ros_toolboxes/interbotix_xs_toolbox/interbotix_xs_modules/src/interbotix_xs_modules/arm.py

# ROS環境とinterbotix_wsのsetup.shをsourceするためのコマンドを.bashrcに追加
RUN echo "source /opt/ros/noetic/setup.sh" >> /root/.bashrc && \
    echo "source /root/interbotix_ws/devel/setup.sh" >> /root/.bashrc

ENV USER root

RUN git clone https://github.com/tonyzhaozh/act.git  # ACT

# Pythonパッケージのインストール
RUN pip3 install torch torchvision pyquaternion pyyaml rospkg pexpect mujoco dm_control opencv-python matplotlib einops packaging h5py ipython h5py_cache pyrealsense2 pyagxrobots pyserial

RUN mkdir /datasets  # エピソードのデータを保存するためのフォルダ。ホストPC上のディレクトリをマウントする用
# sedコマンドを使用してDATA_DIRを上で作成した/datasetsに変更
RUN sed -i "s|DATA_DIR = os.path.expanduser('~/data')|DATA_DIR = os.path.expanduser('/datasets')|" "/root/interbotix_ws/src/mobile-aloha/aloha_scripts/constants.py"


#asratecのリポジトリをクローン
COPY vsido_bilateral_ros/ /root/interbotix_ws/src/vsido_bilateral_ros/
#ロボットモデルコピー
COPY vsido_bilateral_ros/config/ /root/interbotix_ws/src/interbotix_ros_manipulators/interbotix_ros_xsarms/interbotix_xsarm_control/config/
COPY vsido_bilateral_ros/urdf/ /root/interbotix_ws/src/interbotix_ros_manipulators/interbotix_ros_xsarms/interbotix_xsarm_descriptions/urdf/

# ワークスペースのビルド
RUN /bin/bash -c '. /opt/ros/noetic/setup.sh; /root/interbotix_ws/devel/setup.sh; cd /root/interbotix_ws; catkin_make'