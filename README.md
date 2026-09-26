# one-time setup (only needed the first time you ever use git on this machine)
git config --global user.name "Your Name"
git config --global user.email "your.github.email@example.com"

# clone your new repo to your computer
git clone https://github.com/<your-username>/CT-Simulation-System-Docs.git
cd CT-Simulation-System-Docs

# create the datasheet file, paste the content from section 4 below into it,
# then run:
git add SYSTEM_DATASHEET.md
git commit -m "Add initial system datasheet with CT config parameters"
git push origin main
