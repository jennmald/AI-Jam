# AI-Jam
## BMM project for the 1000 Scientists AI Jam

### Developing a Prediction Model for Sample Alignment at BMM

The Beamline for Materials Measurement (BMM) focuses on materials science applications for areas such as energy, health, and the environment. BMM specializes in x-ray absorption spectroscopy (XAS) techniques at hard x-ray energies as well as x-ray diffraction on thin films. Beamline scientist, Bruce Ravel, has been collecting data about alignment scans for approximately a year. This data includes the date and time of the scan, scan type, and the UID of the Tiled record. The alignment problem arises when a human must visually inspect plots and choose the right alignment point. To address this, we aim to leverage the OpenAI o3-mini model we have a goal to develop a function to predict the interpretation of the alignment scan. This prediction will then be presented to the user, offering an automated solution to the alignment problem.