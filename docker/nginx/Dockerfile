# Nginx butuh isi folder public/ (CSS, JS, gambar) untuk melayani file statis.
# Kita ambil langsung dari image "app" yang sudah di-build.
FROM nginx:1.27-alpine
COPY --from=app /var/www/html/public /var/www/html/public