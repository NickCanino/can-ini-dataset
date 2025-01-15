# CANini: Nominal & Parameterized CAN Bus Attack Dataset for Intrusion Detection Systems

This repository contains a link to the CANini dataset, which is too large to be uploaded directly to GitHub. You can download the entire dataset from the Google Drive Folder in the link below:
[CANini Download](https://drive.google.com/drive/folders/1PRpj1szJDsWvfP7upyny1vBQDeYZDn8f?usp=drive_link)

In search of extensive and detailed analysis of the temporal and data-wise carachteristics of the CAN traffic provided in this dataset? We suggest to read our article [CANini: In-Depth Traffic Analysis for Design and Robustness Testing of DTree-based IDS in Automotive Networking Systems](https://doi.org/TBA)!

## Dataset Description

The CANini dataset includes data gathered from a real vehicle (by the CAN-MIRGU's authors), and a Python script that can augment the variety of benign and attack scenarios (through parameterized generation) in the context of CAN communication networks. Our aim is to provide a dataset to develop Intrusion Detection Systems (IDS) that are able to recognise attacks even if some attack characteristics chenge from the ones used in the training phase of the IDS.

As anticipated, we provide a Py script that will be described later in [Python Script](#python-script). This script has different configuration parameters that allow to manipulate some characteristics of the CAN traffic provided in the dataset files:
   - *Temporal characteristic* can be modified by applying a static shift on the absolute arrival times of the selected CAN frames (e.g., $$t_{shifted} = t_{original} + \delta t$$).
   - *Data-wise characteristic* can be modified by applying an AWGN to each byte of the data field of the selected CAN frames (e.g., $$noisyByte_i = originalByte_i + N(\mu, \sigma)$$).
This way, by manipulating $$\delta t$$, $$\mu$$, and $$\sigma$$ we are able to easily augment the available number of CAN traffic traces.

### Attacks on CAN Networks
Over the years, a variety of cyberattacks over CAN networks have been discovered and demonstrated. Commonly, a benign ECU (Electronic Control Unit) is a legitimate network node, and an attacker ECU is an illegitimate network node inserted into the network. As a reference, **Figure 1** schematically depicts the nature of the known attack types on CAN networks.

![attacks on CAN](images/all_attacks_white.png)
*Figure 1: Examples of known attacks on CAN network, highlighting the order and periodicity of considered frames. ECUs A and B are legitimate, while K is the attacker.*

Each attack has its peculiarity, and its effects on the traffic of the targeted CAN bus:
1. **DoS Attack:** floods the CAN bus with high-priority frames (ID near 0x000), preventing other ECUs from sending lower-priority frames.

2. **Fuzzing Attack:** sends frames with random values in the ID, DLC, and payload fields, overwhelming legitimate frames with randomized traffic or traffic targeting unwanted vehicle states.

3. **Replay Attack:** stores valid frames by monitoring CAN traffic and retransmits them later, creating inconsistency in information.

4. **Spoofing Attack:** requires a preliminary payload analysis of at least one ID. Subsequently, malicious frames with spoofed IDs and payloads are sent to force the vehicle into undesired states.

5. **Suspension Attack:** aims to interrupt the transmission of frames of a target ECU by exploiting the error handling of the CAN protocol or by installing malicious software.

6. **Masquerade Attack:** is performed after suspending the transmission of an ECU. Then, a malicious ECU sends spoofed frames with the same ID, DLC, payload properties, and periodicity, essentially keeping CAN traffic unchanged.


## Repository Organization

![dataset structure](images/dataset_augmentation_v3.png)
*Figure 2: Structure of the repository of our dataset, CANini, which comprehends CAN-MIRGU dataset and our extension with further benign and attack CAN traces.

After downloading the *CANini* folder at the link [CANini Download](https://drive.google.com/drive/folders/1PRpj1szJDsWvfP7upyny1vBQDeYZDn8f?usp=drive_link), inside two main folders and a Python script can be noted. In particular:
   - The ```generate_parameterized_file.py``` file contains the Python script to be used for the generation of the parameterized banign and attack traces. See [Python Script](#python-script) for further details on how to use this script.
   - The ```CAN-MIRGU``` folder contains the homonym dataset with all traffic files improved (w.r.t. the original dataset) by inserting the DLC information for each CAN frame. Also, we converted the files in CSV format (originally in LOG) for ease of use in Python and Matlab scripts.
   - The ```PARAMETERIZED``` folder only contains the *README.txt* file. Its sctructure depicted in **Figure 2** is seamlessly created by the Py script.

### Trace Sample
Every CSV file of the dataset has the same structure in which, the first row is the heading and the remaining ones contain information on each transmitted CAN frame:
   - ```timestamp``` provides the absolute arrival time of the CAN frame in terms of seconds \[ $s$ \], with microsecond resolution \[ $\mu s$ \].
   - ```arbitration_id``` contains the ID, in hexadecimal base, of the CAN frame.
   - ```data_field``` is a string with variable length that depends on the number of data bytes transmitted in the CAN frame. Each data byte is represented by a pair of characters, in hexadecimal base.
   - ```dlc_value``` provides the Data Length Code (DLC) of the CAN frame. This positive integer value corresponds to the number of data bytes (pairs of characters in ```data_field```) in the CAN frame.
   - ```attack``` should be interpreted as a flag, since it can only assume the values ```0``` (if the CAN frame is from a benign ECU) and ```1``` (if the CAN frame is from the attacker ECU).
 
   - Snapshot example:
      | Timestamp           | Arbitration ID | Data Field        | DLC Value | Attack |
      |---------------------|----------------|-------------------|-----------|--------|
      | 1698233010.284626   | 372            | 2F002500EE1E0000  | 8         | 0      |
      | 1698233010.284628   | 381            | 80B0390000128005  | 8         | 0      |
      | 1698233010.285633   | 07F            | 00C3000000000000  | 8         | 1      |
      | 1698233010.285637   | 436            | 00000000          | 4         | 0      |
      | 1698233010.285638   | 595            | 00006BFFF30E0000  | 8         | 0      |
      | 1698233010.286656   | 340            | 1300000400002811  | 8         | 0      |
      | 1698233010.286658   | 485            | 00000000          | 4         | 0      |


      ```csv
      timestamp,arbitration_id,data_field,dlc_value,attack
      1698233010.284626,372,2F002500EE1E0000,8,0
      1698233010.284628,381,80B0390000128005,8,0
      1698233010.285633,07F,00C3000000000000,8,1
      1698233010.285637,436,00000000,4,0
      1698233010.285638,595,00006BFFF30E0000,8,0
      1698233010.286656,340,1300000400002811,8,0
      1698233010.286658,485,00000000,4,0
      ...
      ```

### Attack Trace Files

1. ```<name attack>.csv```
   - CAN traffic trace containing a specific attack, in CSV file format. Attacks (a)-(d) of **Figure 1** are performed by injecting malicious CAN frames in real-time into the vehicle's CAN network. Attacks (e) and (f) of **Figure 1** are simulated modifying some attack or benign trace by hand. 
   
2. ```<name attack>_Param.csv```
   - CAN traffic trace containing a specific attack that has been modified through the [Python Script](#python-script) provided, with the same CSV file format as the input file.

### Benign Trace Files

1. ```<name benign>.csv```
   - CAN traffic trace containing the benign traffic gathered from the real vehicle, in CSV file format.

2. ```<name benign>_Param.csv```
   - CAN traffic trace containing the benign traffic that has been modified through the [Python Script](#python-script) provided, ith the same CSV file format as the input file.

### Metadata

1. ```Attacks_metadata.json```
   - JSON file containing metadata for all the attack traces contained in the unmodified part of the dataset.

### Python Script

1. ```generate_parameterized_file.py```
   - It picks the selected original file (```<name>.csv```), and generates a new file (```<name>_Param.csv```) in the destination ```PARAMETERIZED``` folder with identical path. This way, the two main folders will have the same internal structure.

   #### **Configuration parameters of the script**
     
   - ```Python
     input_relative_path = "CAN-MIRGU/Attack/Real_attacks/Break_and_fog_light_attack.csv"  # Example relative path
     ```
     This variable contains the relative path, w.r.t. the Py script, of the selected input CSV file. From this example path, for instance, the script will create the sub-folders (if missing) and place the new file at path ```"PARAMETERIZED/Attack_Param/Real_attacks_Param/Break_and_fog_light_attack_Param.csv"```.
     ***
     
   - ```Python
     awgn_std_dev = 0   # Example AWGN standard deviation factor
     awgn_mean = 0      # Example AWGN mean factor
     ```
     These two variables control the modification of the data bytes of the selected CAN frames in the selected CSV file. They define the mean and std.dev of the AWGN inserted into each byte, as in equation $$noisyByte_i = originalByte_i + N(\mu, \sigma)$$.
     ***
     
   - ```Python
     temporal_shift = 2 / 1000   # Example temporal shift in seconds
     ```
     This variable control the modification of the absolute arrival time of the selected CAN frames in the selected CSV file. The shift is applied in the order of seconds, and it can be positive or negative ($$t_{shifted} = t_{original} + \delta t$$).
     
   - ```Python
     valid_arbitration_ids = ["164", "4F1"]  # Example arbitration IDs to modify
     ```
     This variable contains the list of the ID(s) in which the temporal and/or data-wise properties have to be modified, w.r.t. the selected input CSV file.
     ***
     
   - ```Python
     modify_all = True   # Modify all rows (if True) or only rows with attack_flag = 1 (if False)
     ```
     This variable is a flag that provides further control over the CAN frames to be manipulated from the selected input CSV file. When ```modify_all==True```, the modification will be applied to all the CAN frames with an ID listed in the ```valid_arbitration_ids```; otherwise, selecting ```modify_all==False``` will limit the modification to only the attack CAN frames (with the field ```attack==1```), independently of their ID.

     IMPORTANT: When the flag ```modify_all==False```, the list defined in ```valid_arbitration_ids``` is bypassed because only the attack frames are modified.
     ***
   
## Notes

- **File Format**: All files are in CSV, JSON, and Python script formats. They can be opened with data management software such as Excel, Python (pandas), or JSON viewers.
- **License**: This work is licensed under the [Creative Commons Attribution 4.0 International (CC BY 4.0) License](https://creativecommons.org/licenses/by/4.0/).
- **Contact**: For any questions or clarifications, feel free to reach out to me via email [nicasio.canino@phd.unipi.it](mailto:nicasio.canino@phd.unipi.it)

## Citation

If you use this dataset in your work, please cite the following articles:

```latex
@article{can_ini_dataset,
  title={{CANini}: In-Depth Traffic Analysis for Design and Robustness Testing of DTree-based IDS in Automotive Networking Systems},
  author={Canino, Nicasio and Dini, Pierpaolo and Mazzetti, Stefano and Rossi, Daniele and Saponara, Sergio},
  journal={IEEE Access},
  year={2025},
  volume={TBA},
  number={TBA},
  pages={TBA},
  publisher={IEEE}
}

@article{can_mirgu_dataset,
  title={{CAN-MIRGU}: A Comprehensive CAN Bus Attack Dataset from Moving Vehicles for Intrusion Detection System Evaluation},
  author={Rajapaksha, Sampath and Madzudzo, Garikayi and Kalutarage, Harsha and Petrovski, Andrei and Al-Kadri, M Omar},
  booktitle={Symposium on Vehicles Security and Privacy. Internet Society},
  year={2024}
}
