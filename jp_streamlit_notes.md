## JP Streamlit Notes

Install Streamit with:  
`pip install streamlit`

Run Streamlit with:  
`streamlit hello`
`streamlit run <file>`

For better performance, install the Watchdog module:  
`xcode-select --install`
`pip install watchdog`


Global config:
In a global config file at `~/.streamlit/config.toml` for macOS/Linux or `%userprofile%/.streamlit/config.toml` for Windows:

https://docs.streamlit.io/library/advanced-features/configuration


Run Picks Pool DEV with:
```bash
conda activate streamlit1
cd /Users/jpw/Dropbox/Data_Science/projects/github_projects/2021-11-10_nfl_picks_pool_streamlit/
streamlit run scripts/nfl_picks_pool_streamlit.py
```


