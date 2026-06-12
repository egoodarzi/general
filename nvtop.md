Build a newer nvtop
sudo apt install git cmake build-essential libdrm-dev libsystemd-dev \
                 libudev-dev libncurses5-dev libncursesw5-dev

git clone https://github.com/Syllo/nvtop.git
cd nvtop

mkdir build
cd build

cmake ..
make -j$(nproc)

./src/nvtop

Before installing system-wide, test the locally built binary. If it runs without crashing, then:

sudo make install
