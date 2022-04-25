## Requirement

- battery ảnh hướng đến việc có chơi được trận đấu hay không? thuộc về `match result` context.

- 1 body has 50 batteries, có thể đổi trong tương lai.

- Tham gia 1 battle, giảm 1 battery.

- Kéo 1 body có battery = 50 batteries => body's battery is full.

- Hồi 1 battery = 864s; 50 batteries = 12 hours

- Body đang charge, < 50 batteries, user click stop.

- Body đang charge thì sẽ không assemble được hay đem đi battle được.

- List lên margic eden: Dừng quá trình sạc của body (hold pin tại thời điểm dừng), Không hiển thị body trong Inventory và Chargind Station.

- Hiện tại đang support 1 slot charge, có thể mở rộng lên 3 hoặc nhiều hơn trong tương lai.

- Chỉ có Part type Body mới được charge.

## Requirement techniques

1. Các query mà hệ thống cần đảm bảo:

- Có slot trống cho user charge được hay không?

- Battery hiện tại của 1 body?

- Part Body đó có đang được charge hay không? Nếu có thì thông số charge là gì?

- Part Body đó có đang ở trên Dock charge hay không?

2. Note:

- Database phải nhất quán:

  - Không cho phép 2 body cùng charge 1 lúc
  - Không cho phép 2 body cùng on dock ở 1 slot index
  - Body đang được charge thì chắc chắn phải on dock nhưng Body on dock thì chưa chắc đã được charge

- Current Battery Value không được âm

- Sử dụng virtual column cho các trường (https://pietrzakadrian.com/blog/virtual-column-solutions-for-typeorm) hoặc virtual type cho các dữ liệu không lưu vào trong database, https://stackoverflow.com/questions/64985390/best-practice-for-virtual-computed-columns-in-nest-js-typeorm-openapi

## GRAPHQL TYPE

type PartBatteryEntity {
partId: Int!
userId: Int!
isCharging: Bool!
value: Int!
maximumValue: Int!
startTime: Date!
endTime: Date!
}

type ChargingSlotManagementEntity {
index: Int!
userId: Int!
partId: Int!
}

PartEntity {
...field_from_asset_service

battery: RobotPartBattery
chargingSlot: ChargingSlotManagementEntity
}

## GRAPHQL QUERY

- Modify API partsByUserId
- Modify API chargingPart
- Modify API updateChargingSlot (bây giờ đang là updateStatusDock)

## GRAPHQL MUTATION

mutation {
chargeBodyPart(partId: Int): PartBatteryEntity
unplugBodyPart(partId: Int): PartBatteryEntity
updateChargingSlot(partId: Int, status: Bool): ChargingSlotManagementEntity
}

## DATABASE SCHEMA

1. Refactor bảng game_asset trong DB Game hiện tại thành bảng part_battery:

| Nom           | Type   | Description                                                        | Example                 |
| ------------- | ------ | ------------------------------------------------------------------ | ----------------------- |
| id            | number | unique id                                                          | 1, 2, 3 ...             |
| part_id       | number | part id                                                            | 123, 456, ...           |
| user_id       | string | part id sở hữu của user id nào                                     | 'user123', ...          |
| is_charging   | bool   | lưu giữ state charge của part                                      | true , false            |
| value         | number | current battery value (không được âm, không lớn hơn maximum_value) | 1, 2, 3 ...             |
| maximum_value | number | giá trị tối đa mà value có thể đạt được, default = 50; không âm    | 1, 2, 3 ...             |
| start_time    | date   | thời điểm bắt đầu charge                                           | 2022-04-15 09:23:08.106 |
| end_time      | date   | thời điểm charged full                                             | 2022-04-15 09:23:08.106 |
| created_at    | date   | thời gian bản ghi được thêm vào DB                                 | 2022-03-29 17:47:07.840 |
| updated_at    | date   | thời gian bản ghi được modify                                      | 2022-03-29 17:47:07.840 |

2. ChargingSlot - thêm bảng mới trong DB Game: charging_slot_management

| Nom        | Type   | Description                                                 | Exemple                 |
| ---------- | ------ | ----------------------------------------------------------- | ----------------------- |
| id         | number | unique id                                                   | 1, 2, 3 ...             |
| index      | number | số thứ tự của slot                                          | 1, 2, 3 ...             |
| user_id    | string | Charging Slot sở hữu của user id nào                        | 'user123', ...          |
| part_id    | number | part id nào đang được charge trên slot này, -1 là available | -1, 123, 456, ...       |
| created_at | date   | thời gian bản ghi được thêm vào DB                          | 2022-03-29 17:47:07.840 |
| updated_at | date   | thời gian bản ghi được modify                               | 2022-03-29 17:47:07.840 |
