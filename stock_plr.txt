#!/bin/bash

mkdir -p ~/gcode_files/plr
filepath=$(sed -n "s/.*filepath *= *'\([^']*\)'.*/\1/p" /home/mks/klipper_config/saved_variables.cfg)
filepath=$(printf "$filepath")
echo "$filepath"

last_file=$(sed -n "s/.*last_file *= *'\([^']*\)'.*/\1/p" /home/mks/klipper_config/saved_variables.cfg)
last_file=$(printf "$last_file")
echo "$last_file"
plr=$last_file
echo "plr=$plr"
PLR_PATH=~/gcode_files/plr

file_content=$(cat "${filepath}" | awk '/; thumbnail begin/{flag=1;next}/; thumbnail end/{flag=0} !flag' | grep -v "^;simage:\|^;gimage:")

echo "$file_content" | sed  's/\r$//' | awk -F"Z" 'BEGIN{OFS="Z"} {if ($2 ~ /^[0-9]+$/) $2=$2".0"} 1' > /home/mks/plrtmpA.$$
sed -i 's/Z\./Z0\./g' /home/mks/plrtmpA.$$
#echo "$file_content" | sed 's/\r$//' | awk -F"Z" 'BEGIN{OFS="Z"} {if ($2 ~ /^[0-9]+$/) $2=$2".0"; else if ($2 ~ /^Z[.][0-9]+$/) $2="0"$2} 1' > /home/mks/plrtmpA.$$
cat /home/mks/plrtmpA.$$ | sed -e '1,/Z'${1}'/ d' | sed -ne '/ Z/,$ p' | grep -m 1 ' Z' | sed -ne 's/.* Z\([^ ]*\).*/SET_KINEMATIC_POSITION Z=\1/p' > ${PLR_PATH}/"${plr}"

echo 'M118 START_TEMPS...' >> ${PLR_PATH}/"${plr}"
cat /home/mks/plrtmpA.$$ | sed '/ Z'${1}'/q' | sed -ne '/\(M104\|M140\|M109\|M190\|M106\)/p' >> ${PLR_PATH}/"${plr}"
cat /home/mks/plrtmpA.$$ | sed -ne '/;End of Gcode/,$ p' | tr '\n' ' ' | sed -ne 's/ ;[^ ]* //gp' | sed -ne 's/\\\\n/;/gp' | tr ';' '\n' | grep material_bed_temperature | sed -ne 's/.* = /M140 S/p' | head -1 >> ${PLR_PATH}/"${plr}"
cat /home/mks/plrtmpA.$$ | sed -ne '/;End of Gcode/,$ p' | tr '\n' ' ' | sed -ne 's/ ;[^ ]* //gp' | sed -ne 's/\\\\n/;/gp' | tr ';' '\n' | grep material_print_temperature | sed -ne 's/.* = /M104 S/p' | head -1 >> ${PLR_PATH}/"${plr}"
cat /home/mks/plrtmpA.$$ | sed -ne '/;End of Gcode/,$ p' | tr '\n' ' ' | sed -ne 's/ ;[^ ]* //gp' | sed -ne 's/\\\\n/;/gp' | tr ';' '\n' | grep material_bed_temperature | sed -ne 's/.* = /M190 S/p' | head -1 >> ${PLR_PATH}/"${plr}"
cat /home/mks/plrtmpA.$$ | sed -ne '/;End of Gcode/,$ p' | tr '\n' ' ' | sed -ne 's/ ;[^ ]* //gp' | sed -ne 's/\\\\n/;/gp' | tr ';' '\n' | grep material_print_temperature | sed -ne 's/.* = /M109 S/p' | head -1 >> ${PLR_PATH}/"${plr}"


BG_EX=`tac /home/mks/plrtmpA.$$ | sed -e '/ Z'${1}'[^0-9]*$/q' | tac | tail -n+2 | sed -e '/ Z[0-9]/ q' | tac | sed -e '/ E[0-9]/ q' | sed -ne 's/.* E\([^ ]*\)/G92 E\1/p'`
# If we failed to match an extrusion command (allowing us to correctly set the E axis) prior to the matched layer height, then simply set the E axis to the first E value present in the resemued gcode.  This avoids extruding a huge blod on resume, and/or max extrusion errors.
if [ "${BG_EX}" = "" ]; then
 BG_EX=`tac /home/mks/plrtmpA.$$ | sed -e '/ Z'${1}'[^0-9]*$/q' | tac | tail -n+2 | sed -ne '/ Z/,$ p' | sed -e '/ E[0-9]/ q' | sed -ne 's/.* E\([^ ]*\)/G92 E\1/p'`
fi
M83=$(cat /home/mks/plrtmpA.$$ | sed '/ Z'${1}'/q' | sed -ne '/\(M83\)/p')
if [ -n "${M83}" ];then
 echo 'G92 E0' >> ${PLR_PATH}/"${plr}"
 echo ${M83} >> ${PLR_PATH}/"${plr}"
else
 echo ${BG_EX} >> ${PLR_PATH}/"${plr}"
fi
echo 'G91' >> ${PLR_PATH}/"${plr}"
echo 'G1 Z10' >> ${PLR_PATH}/"${plr}"
echo 'G90' >> ${PLR_PATH}/"${plr}"
echo 'G28 X Y' >> ${PLR_PATH}/"${plr}"
echo 'G1 X5' >> ${PLR_PATH}/"${plr}"
echo 'G1 Y5' >> ${PLR_PATH}/"${plr}"
echo 'G91' >> ${PLR_PATH}/"${plr}"
echo 'G1 Z-5' >> ${PLR_PATH}/"${plr}"
echo 'G90' >> ${PLR_PATH}/"${plr}"
echo 'M106 S204' >> ${PLR_PATH}/"${plr}"

#tac /home/mks/plrtmpA.$$ | sed -e '/ Z'${1}'[^0-9]*$/q' | tac | tail -n+2 | sed -ne '/ Z/,$ p' >> ${PLR_PATH}/"${plr}"
first_line=$(cat /home/mks/plrtmpA.$$ |sed -e '1,/Z'${1}'/ d' | sed -ne '/ Z/,$ p' | grep -m 1 ' Z' | grep -E 'F[0-9]+' | sed -E 's/F[0-9]+/F3000/g')
if [ "${first_line}" = "" ];then
    cat /home/mks/plrtmpA.$$ | sed -e '1,/Z'${1}'/ d' | sed -ne '/ Z/,$ p' >> ${PLR_PATH}/"${plr}"
else
    echo ${first_line} >> ${PLR_PATH}/"${plr}"
    cat /home/mks/plrtmpA.$$ | sed -e '1,/Z'${1}'/ d' | sed -ne '/ Z/,$ p' | tail -n +2 >> ${PLR_PATH}/"${plr}"
fi
rm /home/mks/plrtmpA.$$
