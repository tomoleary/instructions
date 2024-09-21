# Instructions for toggling between `x86` and `arm` `conda` on OSX with ARM chip using `bash` in iTerm2.

Adapted from (this page, which is for `zsh` using Terminal)[https://teamdynamix.umich.edu/TDClient/47/LSAPortal/KB/ArticleDet?ID=10245]. 

This is for those that don't want to use `zsh` or OSX native Terminal. There are not many things that need to be changed from the above linked tutorial.


## Install the x86 translator via Rosetta
```
softwareupdate --install-rosetta
```

## Setting up iTerm2 profile.

1. Open iTerm2

2. In the dropdown menu, click Profiles -> Open Profiles

3. Click on "Edit Profiles". My goal is to have the x86 be the default profile, so I rename the default profile to "x86". In the settings for the default profile, there is a space to enter a command that is executed when the profile launches. In this space enter `arch -x86_64 bash`.

4. Create another profile for "arm". Since arm is the default architecture there is no need to enter a command in the settings for this profile. Customize the  different color scheme for this profile in order to visually distinguish this from the "x86" profile. 


## Installing the different condas

We will use `$HOME/miniconda3` for arm and `$HOME/miniconda_x86` for x86.

### arm

```
curl -O https://repo.anaconda.com/miniconda/Miniconda3-latest-MacOSX-arm64.sh
chmod ug+x Miniconda3-latest-MacOSX-arm64.sh
./Miniconda3-latest-MacOSX-arm64.sh
```
When prompted for install location enter `$HOME/miniconda3`.

When prompted to initialize: `no`.

### x86 

```
curl -O https://repo.anaconda.com/miniconda/Miniconda3-latest-MacOSX-x86_64.sh
chmod ug+x Miniconda3-latest-MacOSX-x86_64.sh
./Miniconda3-latest-MacOSX-x86_64.sh
```
When prompted for install location enter `$HOME/miniconda3_x86`.

When prompted to initialize: `no`.

## Configure `conda` init

We will instruct `conda` to toggle the version based on which architecture is loaded in our shell

```
cd
mkdir .custrc
vi .custrc/.condarc
```

Paste the following in `.condarc`:


```
init_conda() {
   # >>> conda initialize >>>
   conda_path_m1="$HOME/miniconda3"
   __conda_setup="$("${conda_path_m1}/bin/conda" 'shell.bash' 'hook' 2> /dev/null)"
   if [ $? -eq 0 ]; then
      eval "$__conda_setup"
   else
      if [ -f "${conda_path_m1}/etc/profile.d/conda.sh" ]; then
          . "${conda_path_m1}/etc/profile.d/conda.sh"
      else
          export PATH="${conda_path_m1}/bin:$PATH"
      fi
   fi
   unset __conda_setup
   # <<< conda initialize <<<
}

init_conda_intel() {
   # >>> conda initialize >>>
   conda_path_intel="$HOME/miniconda3_x86"
   __conda_setup="$("${conda_path_intel}/bin/conda" 'shell.bash' 'hook' 2> /dev/null)"
   if [ $? -eq 0 ]; then
      eval "$__conda_setup"
   else
      if [ -f "${conda_path_intel}/etc/profile.d/conda.sh" ]; then
          . "${conda_path_intel}/etc/profile.d/conda.sh"
      else
          export PATH="${conda_path_intel}/bin:$PATH"
      fi
   fi
   unset __conda_setup
   # <<< conda initialize <<<
}

```




Then we update our `.bashrc`, to source the custom `.condarc`



```
# init conda based on architecture
source ~/.custrc/.condarc
if [ "$(uname -m)" = "x86_64" ]; then
    init_conda_intel
    echo "conda x86_64 is activated"
    # uncomment following line if using homebrew
    # eval "$(/usr/local/bin/brew shellenv)"
else
    init_conda
    echo "conda m1 is activated"
    # uncomment following line if using homebrew
    # eval "$(/opt/homebrew/bin/brew shellenv)"
fi

```


## Using `conda`.

The conda that is loaded should be based on the architecture tied to your shell profile. So booting up a new session in either profile should load the correct corresponding `conda`. This can be verified with `which conda`.




